# Kubernetes ACD

## Description

This Module provisions a Kubernetes-Cluster using Forman ACD.
Currently, only Calico and Cilium are available as Container Network Interfaces.
By default, Kubernetes-Dashboard is set up but not made available via any ingress rules.

The firewall is configured to allow all in-cluster communication as well as inbound traffic on the ports specified via Ansible group_vars.

## Requirements

This playbook currently expects hosts to run on Ubuntu 20.04.

## Architecture
![](./architecture.svg)

A minimal Kubernetes cluster would consist of a bootstrap node as well as a single worker node and no proxy.
In general, a cluster should consist of as many workers as needed, a proxy node for the API and bootstrap+controlplane
should be an odd number. For example 1 haproxy, 10 worker, 1 bootstrap, 4 controlplane nodes.


## Configuring and Accessing the Cluster
It is important to specify a bootstrap node during application creation.
The node will be treated like any other controlplane node in the cluster, but is used to create the cluster initially.
The bootstrap node should never be removed from the cluster.
Also, this node will have management utilities like kubectl and helm installed.
After deployment, you should be able to locate the Kubernetes admin.conf on the bootstrap node.
For convenience, it has been symlinked to /root/.kube/config.
The Ansible log will contain information about which node has been your bootstrap node at the beginning and end of execution.

As the kubernetes-dashboard is not exposed by default use `kubectl proxy` after retrieving the admin config.
You can now access your dashboard at
[](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)

### Variables

The following host-variables can be set in Orcharhino:

`group_vars/bootstrap/main.yaml`:

| Name | Description | Default | Mandatory (y/n) |
| ---- | ----------- | ------- | --------------- |
| kube_network_plugin | CNI to be installed. Valid values: cilium, calico | "cilium" | y |
| setup_dashboard | Deploy [kubernetes-dashboard](https://github.com/kubernetes/dashboard) to the cluster | false | y |


`group_vars/controlplane/main.yaml`:

| Name | Description | Default | Mandatory (y/n) |
| ---- | ----------- | ------- | --------------- |
| firewall_allow_tcp | Comma separated list of TCP ports to allow from anywhere | "6443,22" | y |
| firewall_allow_udp | Comma separated list of UDP ports to allow from anywhere | "" | y |

`group_vars/worker/main.yaml`:

| Name | Description | Default | Mandatory (y/n) |
| ---- | ----------- | ------- | --------------- |
| firewall_allow_tcp | Comma separated list of TCP ports to allow from anywhere | "22,80,443" | y |
| firewall_allow_udp | Comma separated list of UDP ports to allow from anywhere | "" | y |

`group_vars/haproxy/main.yaml`:

| Name | Description | Default | Mandatory (y/n) |
| ---- | ----------- | ------- | --------------- |
| firewall_allow_tcp | Comma separated list of TCP ports to allow from anywhere | "6443,22" | y |
| firewall_allow_udp | Comma separated list of UDP ports to allow from anywhere | "" | y |

## Warning

This repository is provided as is and not recommended for production use. **Use it at your own risk.**

## Documentation

[Application Centric Deployment Guide](https://docs.orcharhino.com/or/docs/sources/guides/application_centric_deployment.html)
in the Orcharhino documentation.
