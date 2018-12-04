+++
title = "What is Contivpp.io?"
type = "concepts"
summary = "Contivpp.io is a Kubernetes CNI plugin that employs a programmable CNF vSwitch composed of an FD.io/VPP data-plane and VPP agent for feature-rich, high-performance cloud native networking."
weight="1"
image = "/img/concepts/what-is-pteropusio.png"
+++



# Contivpp.io Kubernetes Network Plugin

## Problem Statement

Cloud native interest and uptake is growing fast. With that comes the need for networking to keep pace, in particular where pod-to-pod and pod-t0 ervice connectivity is concerned. Yes the Kubernetes CNI defining an API for managing cluster pod networks is there. An there are multiple tested and deployed [Container Network Interface (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) network plugins enabling pods on the same or different hosts to talk with each other. One could say that Kubernetes Pod Networking 1.0 is complete.

What is needed for K8s Pod Networking 2.0?

- Policies and Services are made first-class citizens. 1.0 addresses connectivity. We need policy and service awareness at the network layer in 2.0 for Pod-to-Service connectivity.

- Performance. Sufficient in 1.0. Cloud native deployment expansion coupled with application workloads sensitive to network resources constraints is a mandatory. This includes any/all data-plane features to realize this objective. Examples inclue such ACLs, NAT, IPsec and tunnel encap/decap.

- Interface Flexibility. 2.0 must be interface versatile or in other words accomodate your classic kernel interfaces (e.g. tap, veth) and newer, performance optimized interfaces such as memif.  

- Observability. There are never enough pushed or pulled stats, telemetry, logs, tracing etc. data to be consumed by management, collection and analytics apps. 2.0 must include mechananisms to needed for detailed awareness and visibility of network behavior and performance.  

- API Support. The K8s API is ideal for K8s cluster management but somewhat limited for Pod networking 1.0. Existing CNI plugins incorprate their own API implementation leading to lock-in dependences. 2.0 needs a common suite of API development tools that allow cloud native and CNI developers to devote cycles to what is really important: applications.

- CNF. Pod Networking 2.0 must embrace cloud native network functions where appropiate. This means introducing CNFs into a k8s cluster dataplane and/or to construct a service chain. 

- Immutablity.     

- Evolvability. K8s Pod Networking 2.0 must innovation autonomy from the existing environment while not deviating from k8s-based deployment and operations best practicies.


It goes without saying that Pod Networking 2.0 must be 100% Kubernetes compliant and be private, hybrid and public friendly. In addition it is ideal to utitlize existing open source project software to build and deploy solutions. 

## Contivpp.io Solution       

Contivpp.io is a Kubernetes CNI network plugin designed and built to address the requirements. It uses [FD.io VPP](https://fd.io/)
to provide policy and service-aware dataplane functionality for k8s cluster networks. The important new functions are

- Maps k8s policy and service mechanisms (i.e. label selectors) to FD.io/VPP functionality. In effect, said policies and services are network-aware leading to the ability for containerized applications to benefit from 

- Implemented in user space for rapid innovation and restartability

- Exposes FD.io/VPP dateplane,  

It deploys itself as a set of system PODs on the master node, and some
of them, in particular on each node in the cluster. On the former are 

![contivpp.io Architecture](/img/what-is-contiv-vpp/contivpp-overview-pict3.png)

Contivpp.io is fully integrated with k8s via its components,
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


|   |   |   |   |   |
|---|---|---|---|---|
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |

