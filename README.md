# ansible-kubernetes-centos7
Ansible play for creating a bare-metal Kubernetes cluster on CentOS 7

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

- A [Flannel](https://github.com/coreos/flannel) overlay network (using default network 10.244.0.0/16 for pods)
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
This will revert (almost) all changes done by the play and deinstall all Kubernetes components from the *master* and *worker* nodes.

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

Additionally, if you intend to use the *glusterfs* feature, you must provide a dedicated block device on all ```kubernetes-worker``` node. The name of this device should be similar on each node.

## Configuration

Most of the global configuration is done via hostgroup variables, in ```group_vars/kubernetes-cluster.example```. Rename this file to ```group_vars/kubernetes-cluster``` and edit it to reflect your desired setup. It will add a global dict object named ```kubernetes``` to your Ansible variables namespace, so don't use that name anywhere else when working with the configured hosts.

### Configure worker nodes

Additionally to the Ansible inventory set-up, the cluster's worker nodes need to be defined in ```kubernetes.cluster.worker_nodes```. You need to specify the FQDN and the (primary) IP addresses of the cluster nodes.

### Feature toggles
The following are boolean values that specify what features you want to be enabled during setup

- ```kubernetes.cluster.feature_toggles.flannel``` defines whether you want a Flannel network setup in your cluster
- ```kubernetes.cluster.feature_toggles.metallb``` defines whether you want a MetalLB setup in your cluster
- ```kubernetes.cluster.feature_toggles.glusterfs``` defines whether you want a GlusterFS setup in your cluster with Heketi provisioning

