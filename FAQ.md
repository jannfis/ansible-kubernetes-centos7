# Frequently Asked Questions

## Why not kubespray?

kubespray is too complex for my needs. The major goal of *ansible-kubernetes-centos7*
is to be as simple as possible, and just for a very specific scenario. Also, I felt
that kubespray is a little too invasive. It does things that I feel belong
somewhere else in your overall Ansible setup and bootstrapping procedures.

## What OS are supported?

CentOS 7 on amd64 (and probably RHEL7 too).

## What is the minimum required Ansible version?

Tested with Ansible 1.7

## Why only Flannel?

Because it is the simplest network overlay. Maybe I'll replace it with a more
advanced one like Calico soon to support network isolation.


