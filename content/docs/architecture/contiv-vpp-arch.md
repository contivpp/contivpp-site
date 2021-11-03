+++
title = "Contivpp.io Architecture"
type = "architecture"
summary = "Contivpp.io Architecture"
+++

## Overview

Contiv-VPP is a Kubernetes network plugin that uses FD.io VPP to provide network connectivity between PODs in a K8s cluster. You deploy it as a set of system PODs in the kube-system namespace, some of them (contiv-ksr, contiv-crd, contiv-etcd) on the control plane node, and some of them (contiv-cni, contiv-vswitch, contiv-stn) on each node in the cluster.

Contiv-VPP is fully integrated with K8s and automatically reprograms itself upon each change passed to the cluster over the K8s API.

The main component of the solution, VPP, runs within the contiv-vswitch POD on each node in the cluster. It provides POD-to-POD, host-to-POD and outside-to-POD connectivity. It leverages VPP's fast data processing that runs in userspace and uses DPDK for direct access to the network I/O layer.

Contiv-VPP reflects K8s services and policies into the VPP configuration. VPP supports services and policies without the need for forwarding packets into the Linux kernel network stack used by kube-proxy. 

---

## Architecture

Contiv-VPP is composed of several components, each packaged and shipped as
a Docker container. 

The components that you run on the control plane node consist of the following:

 - [Contiv KSR](#contiv-ksr)
 - [Contiv CRD + Netctl](#Contiv-CRD-netctl)
 - [Contiv ETCD](#contiv-etcd)
- [Contivpp UI](#contivpp-UI)


The components that you run on worker nodes consist of the following: 

- [Contiv vSwitch](#contiv-vswitch)
- [Contiv CNI](#contiv-cni)
- [Contiv STN](#contiv-stn)


---

This section provides an architecture diagram along with a brief description of the components.


![contivpp arch](/img/what-is-contiv-vpp/contivpp-arch-new4.drawio.svg)
<p style="text-align: center; font-weight: bold">Figure: Contiv-VPP Architecture</p>

### Contiv KSR
[Kubernetes State Reflector](https://github.com/contiv/vpp/tree/master/cmd/contiv-ksr) (KSR) agent subscribes to K8s control plane, watches K8s resources and 
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
reflected by the KSR agent and VPP configuration data. The contiv-vswitch agents 
access the configuration to restore operation after restarts. The data store also contains internal metadata. 


### Contiv vSwitch
vSwitch is the  primary networking component for provides the connectivity between PODs.
Each node in the cluster runs a vswitch composed of the Contiv-VPP agent and a VPP data plane packaged into a single Docker container.

**VPP** is the data plane software for establishing connectivity between PODs, a host Linux
network stack and a NIC interface controlled by VPP.

- PODs are connected to VPP using TAP interfaces wired between VPP and each POD network namespace.
- Host network stack is connected to VPP using a host-VPP interconnect TAP interface contained in the default network namespace.
- NIC is controlled directly by VPP using DPDK. 
  
Note that a VPP-managed NIC is not visible to the host Linux network stack. K8s control plane communication requires a separate management interface. You will need to install a separate interface or use STN. 
 [STN (Steal The NIC)](https://github.com/contiv/vpp/blob/master/docs/SINGLE_NIC_SETUP.md) deployment must be applied.

**Contiv-VPP Agent** is the control plane piece of the vSwitch container. This agent configures the VPP data plane based on information learned from gained from the Contiv ETCD data store, and requests from Contiv STN. The agent is based on the [Ligato VPP Agent](https://ligato.io/) and includes K8s-specific extensions.   

For communication with VPP, the agent uses VPP binary API messages sent via shared memory using 
[GoVPP](https://wiki.fd.io/view/GoVPP).
For connection with Contiv STN, the agent acts as a GRPC server serving CNI requests 
forwarded from the Contiv CNI.


### Contiv CNI
The Contiv CNI binary implements the [Container Network Interface](https://github.com/contiv/vpp/tree/master/cmd/contiv-cni) 
API. Kubelet executes the binary upon POD creation and deletion. The CNI binary
initiates a gRPC request and forwards it to the Contiv VPP Agent
running on the node. The agent processes the request by wiring or unwiring the container. It replies with a response, which is then forwarded back to Kubelet.

### Contiv STN
The default Contiv-VPP setup requires two network interfaces
per node: one controlled by VPP for data facing PODs and one controlled by the host
network stack for K8s control plane communication. 

If your K8s nodes do not come with two network interfaces, Contiv-VPP can work with just one.
The [Contiv STN (Steal The NIC)](https://github.com/contiv/vpp/blob/master/docs/setup/SINGLE_NIC_SETUP.md) daemon "steals" the NIC and its IP address from the host network stack just before starting
the VPP. STN configures the stolen NIC with the same IP address on both VPP and the host-VPP interconnect TAP interface. 



### Contivpp UI
The Contivpp UI displays your K8s cluster including the Contiv-VPP system pods. You can access the network configuration, namespaces and K8s services mapped to contiv vswitches. This component implements a proxy providing REST APIs to the front-end GUI and per-vswitch APIs to the contiv vswitches deployed in the cluster. You deploy the contivpp UI component as a docker container in both your production and demo systems.

---

## Contiv-VPP System Flows

The figure below is a sequence diagram of Contiv-VPP system flows.

![contivpp arch](/img/what-is-contiv-vpp/contiv-flow.png)
<p style="text-align: center; font-weight: bold">Figure: Contiv-VPP System Flows</p>