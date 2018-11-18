# ansible-kubernetes-centos7
Ansible play for creating a bare-metal Kubernetes cluster on CentOS 7

## What's this?

A collection of Ansible configuration to set up a single master, multi minion cluster on bare metal CentOS 7 (amd64) environments. Running this play, you will end up with a simple but fully working Kubernetes cluster. I have created this mainly for my own entertainment, so your mileage may vary. 

## Features:

- A Flannel overlay network (using default network 10.244.0.0/16 for pods)
- A Metal-LB load balancer using L2 provisioning (configurable provisioning network)
- GlusterFS setup with heketi for automatic provisioning of volumes (requires dedicated storage)
