# kubernetes-node

Install kubernetes on a server node.
Losely based on geerlingguy's approach, but entirely rewritten, fixed in some places, and expanded on others. 
Can handle controlplane (also HA) and worker nodes.
Can bootstrap a controlplane.

## Requirements

## Role Variables

- `kube_node_role`: Role of the node in the cluster; possible values are
    - `bootstrap`: Bootstrap a cluster on this node and read `kubeadm join` commands
    - `controlplane`: Add this node to the controlplane of an existing cluster
    - `worker`: Add this node to an existing cluster as a worker

Depending on `kube_node_role`, the following variables are **required**:
- `kube_controlplane_endpoint`: Endpoint for the controlplane; defaults to `{{ ansible_host }}:6443`, which works for non HA setups without a loadbalancer; necessary for the `bootstrap` node in an HA setup
- `kube_kubeadm_join_command`: Command to execute in order to join a node to the cluster; necessary for `controlplane` and `worker` nodes; will be set as an ansible fact of the `bootstrap` node (of the same name) when this role is executed on said node
- `kube_certificate_key`: Key to decrypt controlplane certificates when joining a `controlplane`; necessary for `controlplane` nodes in an HA setup; will be set as an ansible fact of the `bootstrap` node (of the same name) when this role is executed on said node


The following variables are optional:
- `kube_version`: Version of kubernetes to install, defaults to 1.22
- `kube_kubeadm_init_extra_opts`: Extra options for `kubeadm init`; relevant for the `bootstrap` node
- `kube_kubeadm_join_extra_opts`: Extra options for `kubeadm join`; relevant for `controlplane` and `worker` nodes
- `kube_advertise_address`: IP address of the controlplane nodes to use for connections to other nodes; setting this is the easiest way to choose a network interface for Kubernetes; defaults to the IP address of the default interface
- `kube_pod_network_cidr`: IP range for Pods in Calico overlay network; defaults to `192.168.0.0/16`
- `kube_kubeadm_config`: Override the contents of kubeadm's config file; see `defaults/main.yaml` for format
- `kube_calico_manifest_file`: Url of the calico manifest to use

## Dependencies

Requires a preinstalled container runtime; tested with containerd.

## Example Playbook

~~~
- name: Install containerd
  hosts: all
  roles:
    - role: geerlingguy.containerd

- name: Bootstrap controlplane
  hosts: bootstrap
  roles:
    - role: kubernetes-node
      vars:
        kube_node_role: "bootstrap"
        kube_controlplane_endpoint: "load.balancer.io:6443"

- name: Add other controlplane nodes
  hosts: controlplane
  roles:
    - role: kubernetes-node
      vars:
        kube_node_role: "controlplane"
        kube_certificate_key: "{{ hostvars['bootstrap']['kube_certificate_key'] }}"
        kube_kubeadm_join_command: "{{ hostvars['bootstrap']['kube_kubeadm_join_command'] }}"

- name: Add workers
  hosts: worker
  roles:
    - role: kubernetes-node
      vars:
        kube_node_role: "worker"
        kube_kubeadm_join_command: "{{ hostvars['bootstrap']['kube_kubeadm_join_command'] }}"
~~~

## License

GPL 3.0

## Author Information

Pascal Fries

ATIX AG

atix.de orcharhino.de
