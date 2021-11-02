+++
title = "Contivpp.io Architecture"
type = "architecture"
summary = "Contivpp.io Architecture"
+++


## Architecture

Contiv-VPP is composed of several components, each packaged and shipped as
a Docker container. 

The components that you run on the control plane node consist of the following:

 - [Contiv KSR](#contiv-ksr)
 - [Contiv CRD + Netctl](#Contiv-CRD-netctl)
 - [Contiv ETCD](#contiv-etcd)


The components that you run on worker nodes consist of the following: 

- [Contiv vSwitch](#contiv-vswitch)
- [Contiv CNI](#contiv-cni)
- [Contiv STN](#contiv-stn)
- [Contivpp UI](#contivpp-UI)
- [Contivpp System Flow](#contivpp-system-flow)


<strong>Note:</strong>. You can set up your cluster to run the worker node components on the control plane. 

---

This section provides an architecture diagram along with a brief description of the components.


![contivpp arch](/img/what-is-contiv-vpp/contivpp-v2-arch-new.png)
<p style="text-align: center; font-weight: bold">Figure: Contiv-VPP Architecture</p>

### Contiv KSR
[Kubernetes State Reflector]((https://github.com/contiv/vpp/blob/master/cmd/contiv-ksr/README.md)) (KSR) agent subscribes to K8s control plane, watches K8s resources and 
propagates all relevant cluster-related information to the Contiv ETCD data store. 

The other Contiv components don't access the K8s API directly. Instead, they subscribe to the
Contiv ETCD data store. 

### Contiv CRD netctl
The [Contiv CRD](https://github.com/contiv/vpp/blob/master/docs/operation/TOOLS.md#contiv--vpp-custom-resource-definitions-crds) handles K8s custom resource definitions (CRD) defined in the K8s API. The CRDs are converted
into configuration data and put to the Contiv ETCD data store. The data includes the configuration individual K8s nodes such as IP address and default
gateway. The Contiv CRD runs a periodic validation of your topology and exports the results as another CRD entry.

You can use the [contiv-netctl](https://contivpp.io/blog/using-conti-vpp-netctl-blog/) CLI tool to
explore runtime state of the cluster and debug the VPP data plane components in the cluster.


### Contiv ETCD
Contiv-VPP uses its own etcd data store for storing K8s cluster-related data
reflected by the KSR agent and VPP configuratin data. The contiv-vswitch agents running on your
individual nodes access the configuration, mostly to restore operation after restarts. The data store also contains internal metadata. 


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

**Contiv-VPP Agent** is the control plane part of the vSwitch container. It is responsible
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
As already mentioned, the default setup of Contiv-VPP requires 2 network interfaces
per node: one controlled by VPP for data facing PODs and one controlled by the host
network stack for k8s control plane communication. In case that your k8s nodes
do not provide 2 network interfaces, contivpp.io can work in the single NIC setup,
when the interface will be "stolen" from the host network stack just before starting
the VPP and configured with the same IP address on VPP, as well as 
on the host-VPP interconnect TAP interface, as it had in the host before it. 
For more information on STN setup, read the [Single NIC Setup README](https://github.com/contiv/vpp/blob/master/docs/SINGLE_NIC_SETUP.md).


### Contivpp UI
Contivpp UI is composed of two components. The first is a customized GUI enabling the user to display the k8s cluster including the Contiv-VPP system pods. It also allows access to the configuration (e.g. IPAM), k8s, contivpp and namespace and k8s services mapped to contiv vswitches. The other component is a proxy providing REST APIs to the front-end GUI and per-vswitch APIs to the contiv vswitches deployed in the cluster. The contivpp UI is deployed as a docker container and is optional both in the production and demo systems.


### Contivpp Flow

![contivpp arch](/img/what-is-contiv-vpp/contiv-flow.png)