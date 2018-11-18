# ansible-kubernetes-centos7
Ansible play for creating a bare-metal Kubernetes cluster on CentOS 7

## What's this?

A collection of Ansible configuration to set up a single master, multi minion cluster on bare metal CentOS 7 (amd64) environments. Running this play, you will end up with a simple but fully working Kubernetes cluster. I have created this mainly for my own entertainment, so your mileage may vary. 

## Features:

- A Flannel overlay network (using default network 10.244.0.0/16 for pods)
- A Metal-LB load balancer using L2 provisioning (configurable provisioning network)
- GlusterFS setup with heketi for automatic provisioning of volumes (requires dedicated storage)

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

## Configuration

Most of the global configuration is done via hostgroup variables, in ```group_vars/kubernetes-cluster```. It will add a global dict object named ```kubernetes``` to your namespace, so don't use that name anywhere else.
