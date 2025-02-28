# Deploy a Production Ready Kubernetes Cluster

If you have questions, check the documentation at [kubespray.io](https://kubespray.io) and join us on the [kubernetes slack](https://kubernetes.slack.com), channel **\#kubespray**.


## Quick Start

### Ansible

#### Usage

```ShellSession
# Install dependencies from ``requirements.txt``
sudo pip3 install -r requirements.txt

# Copy ``inventory/sample`` as ``inventory/mycluster``
cp -rfp inventory/sample inventory/mycluster

# Update Ansible inventory file with inventory builder
declare -a IPS=(8.208.16.47 8.208.112.61)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# Change cluster node role and ips
sudo vi inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
# Change cluster configuration 
sudo vi inventory/mycluster/hosts.yaml

# Check and update the playbook
sudo vi cluster.yml

# Deploy Kubespray with Ansible Playbook - run the playbook as root
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```


```


## Documents

- [Requirements](#requirements)
- [Kubespray vs ...](docs/comparisons.md)
- [Getting started](docs/getting-started.md)
- [Setting up your first cluster](docs/setting-up-your-first-cluster.md)
- [Ansible inventory and tags](docs/ansible.md)
- [Integration with existing ansible repo](docs/integration.md)
- [Deployment data variables](docs/vars.md)
- [DNS stack](docs/dns-stack.md)
- [HA mode](docs/ha-mode.md)
- [Network plugins](#network-plugins)
- [Vagrant install](docs/vagrant.md)
- [Flatcar Container Linux bootstrap](docs/flatcar.md)
- [Fedora CoreOS bootstrap](docs/fcos.md)
- [Debian Jessie setup](docs/debian.md)
- [openSUSE setup](docs/opensuse.md)
- [Downloaded artifacts](docs/downloads.md)
- [Cloud providers](docs/cloud.md)
- [OpenStack](docs/openstack.md)
- [AWS](docs/aws.md)
- [Azure](docs/azure.md)
- [vSphere](docs/vsphere.md)
- [Packet Host](docs/packet.md)
- [Large deployments](docs/large-deployments.md)
- [Adding/replacing a node](docs/nodes.md)
- [Upgrades basics](docs/upgrades.md)
- [Air-Gap installation](docs/offline-environment.md)
- [Roadmap](docs/roadmap.md)

## Supported Linux Distributions

- **Flatcar Container Linux by Kinvolk**
- **Debian** Buster, Jessie, Stretch, Wheezy
- **Ubuntu** 16.04, 18.04, 20.04
- **CentOS/RHEL** 7, 8 (experimental: see [centos 8 notes](docs/centos8.md))
- **Fedora** 32, 33
- **Fedora CoreOS** (experimental: see [fcos Note](docs/fcos.md))
- **openSUSE** Leap 15.x/Tumbleweed
- **Oracle Linux** 7, 8 (experimental: [centos 8 notes](docs/centos8.md) apply)

Note: Upstart/SysV init based OS types are not supported.

## Supported Components

- Core
  - [kubernetes](https://github.com/kubernetes/kubernetes) v1.20.6
  - [etcd](https://github.com/coreos/etcd) v3.4.13
  - [docker](https://www.docker.com/) v19.03 (see note)
  - [containerd](https://containerd.io/) v1.4.4
  - [cri-o](http://cri-o.io/) v1.19 (experimental: see [CRI-O Note](docs/cri-o.md). Only on fedora, ubuntu and centos based OS)
- Network Plugin
  - [cni-plugins](https://github.com/containernetworking/plugins) v0.9.0
  - [calico](https://github.com/projectcalico/calico) v3.16.9
  - [canal](https://github.com/projectcalico/canal) (given calico/flannel versions)
  - [cilium](https://github.com/cilium/cilium) v1.8.8
  - [flanneld](https://github.com/coreos/flannel) v0.13.0
  - [kube-ovn](https://github.com/alauda/kube-ovn) v1.6.1
  - [kube-router](https://github.com/cloudnativelabs/kube-router) v1.2.0
  - [multus](https://github.com/intel/multus-cni) v3.7.0
  - [ovn4nfv](https://github.com/opnfv/ovn4nfv-k8s-plugin) v1.1.0
  - [weave](https://github.com/weaveworks/weave) v2.8.1
- Application
  - [ambassador](https://github.com/datawire/ambassador): v1.5
  - [cephfs-provisioner](https://github.com/kubernetes-incubator/external-storage) v2.1.0-k8s1.11
  - [rbd-provisioner](https://github.com/kubernetes-incubator/external-storage) v2.1.1-k8s1.11
  - [cert-manager](https://github.com/jetstack/cert-manager) v0.16.1
  - [coredns](https://github.com/coredns/coredns) v1.7.0
  - [ingress-nginx](https://github.com/kubernetes/ingress-nginx) v0.43.0

Note: The list of available docker version is 18.09, 19.03 and 20.10. The recommended docker version is 19.03. The kubelet might break on docker's non-standard version numbering (it no longer uses semantic versioning). To ensure auto-updates don't break your cluster look into e.g. yum versionlock plugin or apt pin).

## Requirements

- **Minimum required version of Kubernetes is v1.18**
- **Ansible v2.9.x, Jinja 2.11+ and python-netaddr is installed on the machine that will run Ansible commands, Ansible 2.10.x is not supported for now**
- The target servers must have **access to the Internet** in order to pull docker images. Otherwise, additional configuration is required (See [Offline Environment](docs/offline-environment.md))
- The target servers are configured to allow **IPv4 forwarding**.
- If using IPv6 for pods and services, the target servers are configured to allow **IPv6 forwarding**.
- The **firewalls are not managed**, you'll need to implement your own rules the way you used to.
    in order to avoid any issue during deployment you should disable your firewall.
- If kubespray is ran from non-root user account, correct privilege escalation method
    should be configured in the target servers. Then the `ansible_become` flag
    or command parameters `--become or -b` should be specified.

Hardware:
These limits are safe guarded by Kubespray. Actual requirements for your workload can differ. For a sizing guide go to the [Building Large Clusters](https://kubernetes.io/docs/setup/cluster-large/#size-of-master-and-master-components) guide.

- Master
  - Memory: 1500 MB
- Node
  - Memory: 1024 MB

## Network Plugins

You can choose between 10 network plugins. (default: `calico`, except Vagrant uses `flannel`)

- [flannel](docs/flannel.md): gre/vxlan (layer 2) networking.

- [Calico](https://docs.projectcalico.org/latest/introduction/) is a networking and network policy provider. Calico supports a flexible set of networking options
    designed to give you the most efficient networking across a range of situations, including non-overlay
    and overlay networks, with or without BGP. Calico uses the same engine to enforce network policy for hosts,
    pods, and (if using Istio and Envoy) applications at the service mesh layer.

- [canal](https://github.com/projectcalico/canal): a composition of calico and flannel plugins.

- [cilium](http://docs.cilium.io/en/latest/): layer 3/4 networking (as well as layer 7 to protect and secure application protocols), supports dynamic insertion of BPF bytecode into the Linux kernel to implement security services, networking and visibility logic.

- [ovn4nfv](docs/ovn4nfv.md): [ovn4nfv-k8s-plugins](https://github.com/opnfv/ovn4nfv-k8s-plugin) is the network controller, OVS agent and CNI server to offer basic SFC and OVN overlay networking.

- [weave](docs/weave.md): Weave is a lightweight container overlay network that doesn't require an external K/V database cluster.
    (Please refer to `weave` [troubleshooting documentation](https://www.weave.works/docs/net/latest/troubleshooting/)).

- [kube-ovn](docs/kube-ovn.md): Kube-OVN integrates the OVN-based Network Virtualization with Kubernetes. It offers an advanced Container Network Fabric for Enterprises.

- [kube-router](docs/kube-router.md): Kube-router is a L3 CNI for Kubernetes networking aiming to provide operational
    simplicity and high performance: it uses IPVS to provide Kube Services Proxy (if setup to replace kube-proxy),
    iptables for network policies, and BGP for ods L3 networking (with optionally BGP peering with out-of-cluster BGP peers).
    It can also optionally advertise routes to Kubernetes cluster Pods CIDRs, ClusterIPs, ExternalIPs and LoadBalancerIPs.

- [macvlan](docs/macvlan.md): Macvlan is a Linux network driver. Pods have their own unique Mac and Ip address, connected directly the physical (layer 2) network.

- [multus](docs/multus.md): Multus is a meta CNI plugin that provides multiple network interface support to pods. For each interface Multus delegates CNI calls to secondary CNI plugins such as Calico, macvlan, etc.

The choice is defined with the variable `kube_network_plugin`. There is also an
option to leverage built-in cloud provider networking instead.
See also [Network checker](docs/netcheck.md).

## Ingress Plugins

- [ambassador](docs/ambassador.md): the Ambassador Ingress Controller and API gateway.

- [nginx](https://kubernetes.github.io/ingress-nginx): the NGINX Ingress Controller.

## Community docs and resources

- [kubernetes.io/docs/setup/production-environment/tools/kubespray/](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)
- [kubespray, monitoring and logging](https://github.com/gregbkr/kubernetes-kargo-logging-monitoring) by @gregbkr
- [Deploy Kubernetes w/ Ansible & Terraform](https://rsmitty.github.io/Terraform-Ansible-Kubernetes/) by @rsmitty
- [Deploy a Kubernetes Cluster with Kubespray (video)](https://www.youtube.com/watch?v=CJ5G4GpqDy0)

## Tools and projects on top of Kubespray

- [Digital Rebar Provision](https://github.com/digitalrebar/provision/blob/v4/doc/integrations/ansible.rst)
- [Terraform Contrib](https://github.com/kubernetes-sigs/kubespray/tree/master/contrib/terraform)

## CI Tests

[![Build graphs](https://gitlab.com/kargo-ci/kubernetes-sigs-kubespray/badges/master/pipeline.svg)](https://gitlab.com/kargo-ci/kubernetes-sigs-kubespray/pipelines)

CI/end-to-end tests sponsored by: [CNCF](https://cncf.io), [Packet](https://www.packet.com/), [OVHcloud](https://www.ovhcloud.com/), [ELASTX](https://elastx.se/).

See the [test matrix](docs/test_cases.md) for details.
