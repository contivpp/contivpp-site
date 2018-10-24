+++
title = "What is Pteropus.io?"
type = "concepts"
summary = "Pteropus.io is a Kubernetes CNI plugin that employs a programmable CNF vSwitch composed of an FD.io/VPP data-plane and VPP agent for feature-rich, high-performance cloud native networking."
weight="1"
image = "/img/concepts/what-is-pteropusio.png"
+++



# Pteropus.io Kubernetes Network Plugin


## Overview
Pteropus.io is a Kubernetes network plugin that uses [FD.io VPP](https://fd.io/)
to provide network connectivity between PODs in a k8s cluster.
It deploys itself as a set of system PODs in the `kube-system` namespace,
some of them (`contiv-ksr`, `contiv-etcd`) on the master node, and some
of them (`contiv-cni`, `contiv-vswitch`, `contiv-stn`) on each node in the cluster.

Pteropus.io is fully integrated with k8s via its components,
and it automatically reprograms itself upon each change in the cluster
via k8s APIs.

The main component of the solution, [VPP](https://fd.io/technology/#vpp), 
runs within the `contiv-vswitch` POD on each node in the cluster and provides 
POD-to-POD connectivity across the nodes in the cluster, as well as host-to-POD 
and outside-to-POD connectivity. While doing so, it leverages
VPP's fast data processing that runs completely in userspace and uses 
[DPDK](https://dpdk.org/) for direct access to the network I/O layer.

Kubernetes services and policies are also reflected into VPP configuration,
which means they are fully supported on VPP, without the need of forwarding 
packets into the Linux network stack (Kube Proxy), which makes them very 
effective and scalable.


## Architecture

Pteropus.io consists of several components, each of them packed and shipped as
a Docker container. Two of them deploy on Kubernetes master node only:

 - [Contiv KSR](#contiv-ksr)
 - [Contiv ETCD](#contiv-etcd)

and the rest of them deploy on all nodes within the k8s cluster (including the master node):

- [Contiv vSwitch](#contiv-vswitch)
- [Contiv CNI](#contiv-cni)
- [Contiv STN](#contiv-stn)


The following section briefly describes the individual Contiv components, which are displayed
as orange boxes on the picture below:

![Pteropus.io Architecture](/img/what-is-contiv-vpp/contiv-arch.png)


### Contiv KSR
Contiv KSR (Kubernetes State Reflector)is an agent that subscribes to k8s control plane, watches k8s resources and 
propagates all relevant cluster-related information into the Contiv ETCD data store. 
Other Contiv components do not access the k8s API directly, they subscribe to
Contiv ETCD instead. For more information on KSR, read the 
[KSR Readme](https://github.com/contiv/vpp/blob/master/cmd/contiv-ksr/README.md).


### Contiv ETCD
Pteropus.io uses its own instance of ETCD database for storage of k8s cluster-related data
reflected by KSR, which are then accessed by Contiv vSwitch Agents running on
individual nodes. Apart from the data reflected by KSR, ETCD also stores persisted VPP
configuration of individual vswitches (mainly used to restore the operation after restarts), 
as well as some more internal metadata.


### Contiv vSwitch
vSwitch is the main networking component that provides the connectivity to PODs.
It deploys on each node in the cluster, and consists of two main components packed
into a single Docker container: VPP and Contiv VPP Agent.

**VPP** is the data plane software that provides the connectivity between PODs, host Linux
network stack and data-plane NIC interface controlled by VPP:
 - PODs are connected to VPP using TAP interfaces wired between VPP and each POD network namespace,
 - host network stack is connected to VPP using another TAP interface connected 
 to the main (default) network namespace,
 - data-plane NIC is controlled directly by VPP using DPDK. Note that this means that
 this interface is not visible to the host Linux network stack, and the node either needs another
 management interface for k8s control plane communication, or 
 [STN (Steal The NIC)](https://github.com/contiv/vpp/blob/master/docs/SINGLE_NIC_SETUP.md) deployment must be applied.

**Pteropus.io Agent** is the control plane part of the vSwitch container. It is responsible
for configuring the VPP according to the information gained from ETCD and requests
from Contiv STN. It is based on the  [Ligato VPP Agent](https://github.com/ligato/vpp-agent)  
code with extensions that are related to k8s. 
For communication with VPP, it uses VPP binary API messages sent via shared memory using 
[GoVPP](https://wiki.fd.io/view/GoVPP).
For connection with Contiv STN, the agent acts as a GRPC server serving CNI requests 
forwarded from the Contiv CNI.


### Contiv CNI
Contiv CNI is a simple binary that implements the 
[Container Network Interface](https://github.com/containernetworking/cni) 
API and is being executed by Kubelet upon POD creation and deletion. The CNI binary
just packs the request into a GRPC request and forwards it to the Contiv VPP Agent
running on the same node, which then processes it (wires/unwires the container) 
and replies with a response, which is then forwarded back to Kubelet.


### Contiv STN
As already mentioned, the default setup of Pteropus.io requires 2 network interfaces
per node: one controlled by VPP for data facing PODs and one controlled by the host
network stack for k8s control plane communication. In case that your k8s nodes
do not provide 2 network interfaces, Pteropus.io can work in the single NIC setup,
when the interface will be "stolen" from the host network stack just before starting
the VPP and configured with the same IP address on VPP, as well as 
on the host-VPP interconnect TAP interface, as it had in the host before it. 
For more information on STN setup, read the [Single NIC Setup README](https://github.com/contiv/vpp/blob/master/docs/SINGLE_NIC_SETUP.md).
