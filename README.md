# ansible-kubernetes-centos7

Ansible play for creating a bare-metal Kubernetes cluster on CentOS 7 from scratch.

## What's this?

A collection of Ansible assets to set up a single master, multi minion cluster on bare metal (or virtualized) CentOS 7 environments. Running this play, you will end up with a simple but fully working Kubernetes cluster. I have created this mainly for my own entertainment, so your mileage may vary.

The play is designed to be as simple as possible, and should be considered only a starting point. Do yourself a favour and do not use this to set up production clusters. It is designed rather modular and contains the following roles that can be reused in own plays:

- *k8s-common*, which executes all common tasks (applicable to all hosts in the cluster)
- *k8s-master*, which executes all system relevant tasks on the cluster master node
- *k8s-worker*, which executes all system relevant tasks on each cluster worker node
- *k8s-cluster*, which executes all Kubernetes cluster tasks (executes on the master node)

It started out from the tutorial at https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-1-10-cluster-using-kubeadm-on-centos-7

## Features:

All features are optional and can be toggled on/off depending on your requirements.

**Supported CNI/networking modules:**
- A [Flannel](https://github.com/coreos/flannel) overlay network (using default network 10.244.0.0/16 for pods)

**Cluster features:**
- A [MetalLB](https://metallb.universe.tf/) load balancer using L2 provisioning (configurable provisioning network)
- A [GlusterFS](https://www.gluster.org/) setup using [Heketi](https://github.com/heketi/heketi) for automatic provisioning of volumes (requires a dedicated partition on each worker node for storage)

## How to use it

1. Clone the repository to your Ansible master node
2. Adapt configuration to your environment (in ```group_vars/kubernetes-cluster```, see below)
3. Create (or adapt) Ansible inventory information (see below)
4. Run ```ansible-playbook``` to execute ```kubernetes-cluster.yml```. The tasks require super user privileges on the remote hosts

## Deinstallation

If you want to deinstall Kubernetes from your nodes completely, you can do so by supplying the variable ```kubernetes_cleanup``` to the play's execution, e.g. as so:

```
$ ansible-playbook -u root kubernetes-cluster.yml -e kubernetes_cleanup=true
```
This will revert (almost) all changes done by the play and deinstall all Kubernetes components from the *master* and *worker* nodes. If you want the nodes to be rebooted after clean up, also use ```-e reboot=true``` as argument.

If you also want to revert what the GlusterFS configuration did (i.e. clean up all storage), specify ```-e cleanup_storage=yes``` as well. Be careful! This will destroy **all** data and partitions on the block device you specified as ```kubernetes.features.glusterfs.device```.

**Attention:** By default, the play will continue with a re-installation after running the clean up tasks. If you just want to cleanup, tell Ansible to only execute tasks tagged ```cleanup```, e.g. by specifying ```--tags cleanup``` on the command line.

## Pre-requisites

Before you begin, the following hostgroups must be defined in your Ansible inventory:

```yaml
# K8S - master node
[kubernetes-master]
k8s-master.example.com

# K8S - minion nodes
[kubernetes-worker]
k8s-node-001.example.com
k8s-node-002.example.com
k8s-node-003.example.com
# ... add more worker nodes to your liking

# K8S - complete cluster
[kubernetes-cluster:children]
kubernetes-master
kubernetes-worker
```
Of course, all the nodes should already be bootstrapped (i.e. user account(s) set up, SSH keys deployed, etc.). These tasks are all beyond the scope of this play.

All nodes configured above should be able to fetch stuff from the Internet via HTTPS, at least through a HTTP proxy server. Offline installations are not supported.

Additionally, if you intend to use the *glusterfs* feature, you must provide a dedicated block device on all ```kubernetes-worker``` node. The name of this device should be similar on each node.

## Configuration

Most of the global configuration is done via hostgroup variables, in ```group_vars/kubernetes-cluster.example```. Rename this file to ```group_vars/kubernetes-cluster``` and edit it to reflect your desired setup. It will add a global dict object named ```kubernetes``` to your Ansible variables namespace, so don't use that name anywhere else when working with the configured hosts.

### Configure worker nodes

Additionally to the Ansible inventory set-up, the cluster's worker nodes need to be defined in ```kubernetes.cluster.worker_nodes```. You need to specify the FQDN and the (primary) IP addresses of the cluster nodes.

### CNI/networking configuration
- Set ```kubernetes.cluster.network.type``` to your desired network overlay plugin (currently, only ```flannel``` is supported).
- Set ```kubernetes.cluster.network.cidr``` to the overlay network you want to allocate addresses from for your pods

### Feature toggles
The following are boolean values that specify what features you want to be enabled during setup

- ```kubernetes.cluster.feature_toggles.metallb``` defines whether you want a MetalLB setup in your cluster
- ```kubernetes.cluster.feature_toggles.glusterfs``` defines whether you want a GlusterFS setup in your cluster with Heketi provisioning

Each feature must then be configured individually in the ```kubernetes.features``` section.

### Kubernetes user
The plays require a non-privileged user for performing the Kubernetes cluster tasks. This user is configured in the ```kubernetes.cluster.k8s_user``` variable. This user can either be managed or unmanaged by this plays. In managed mode, the user and group will be created along with any supplemental configuration (like required SSH keys). Also, in *managed* mode this user will be deleted (along with its home directory!) from all nodes when running cleanup tasks. Likewise, in *unmanaged* mode, this plays will assume an already existing user on the nodes and will not touch it during cleanup.

### Configure the storage cluster
For the GlusterFS cluster to work, you need at least three worker nodes for your cluster. Each of the nodes must have a dedicated partition (or whole disk) available for Gluster to use, and they should ideally be of the same size on all nodes. All configurables are found in the ```kubernetes.features.glusterfs``` map:

- ```namespace``` defines the namespace in your Kubernetes cluster to install the components
- ```device``` is the name of the block device to use for clustered storage (must be same across all nodes!)
- ```storageclass_name``` is optional, and if given, is the name of the StorageClass object created for automatic volume provisioning within the Kubernetes cluster
- ```heketi_endpoint_clusterip``` defines the ClusterIP resource to allocate and bind the Heketi API to. This will be the endpoint you set ```HEKETI_CLI_SERVER``` to when talking with the API.

## Thinks to keep in mind / Caveats
As mentioned above, the Kubernetes cluster set up by this play should not be used as-is for production purposes. 

Some important things to keep in mind:

1. Flannel does not provide any network isolation for your pods whatsoever (all pods can communicate with every other pod in the cluster)
2. Heketi is not set up for authentication, so basically everyone with HTTP access to the API endpoint can perform actions on your storage cluster, up to deleting volumes and even the cluster it self.
3. All certificates in use are self-signed during the setup by kubeadm, you might want to replace them with your own certificates
4. The GlusterFS setup is very default and nil (nada, zil) tuned nor tested

## Feedback, suggestions, pull requests
Yes.
