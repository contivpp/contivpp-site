+++
title = "Brief Description"
type = "concepts"
summary = "Outlines the Problem Statment and several important aspects of the Contivpp.io Solution"
weight = "1"
+++



# Contivpp.io Kubernetes Network Plugin

## Problem Statement

Cloud native interest and uptake is growing fast. With that comes the need for networking to keep pace, in particular where pod-to-pod and pod-to-service connectivity is concerned. Yes the Kubernetes CNI defining an API for managing cluster pod networks is there. And there are multiple tested and deployed [Container Network Interface (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) network plugins enabling pods on the same or different hosts to talk with each other. One could say that Kubernetes Pod Networking 1.0 is complete.

What is needed for K8s Pod Networking 2.0?

- __Policies and Services are made first-class citizens__ which translates to k8s policy and service awareness at the network layer.

- __Performance.__ Cloud native deployment expansion coupled with application workloads sensitive to network resources constraints is a mandatory. This includes any/all data-plane features to realize this objective including but not limited to such ACLs, NAT, IPsec and tunnel encap/decap.

- __Interface Versatility.__ Must accomodate your classic kernel interfaces (e.g. tap, veth) and newer, performance optimized interfaces such as memif.  

- __Observability.__ There are never enough pushed or pulled stats, telemetry, logs, tracing etc. data to be consumed by management, collection and analytics apps. 2.0 must include mechananisms to needed for detailed awareness and visibility of network behavior and performance.  

- __API Support.__ The k8s API is ideal for K8s cluster management but somewhat limited cluster networking. Existing CNI plugins incorprate their own API implementation resulting in potential lock-in dependences. API development tools, plugins and management agents makes it easy to build applications and services to interact with the underlying networking. It also allows cloud native and CNI developers to devote cycles to what is really important: applications.

- __CNF.__ Embrace cloud native network functions where appropiate. This means introducing CNFs into a k8s cluster dataplane and/or to construct a service chain. 

- __Immutablity.__ Must subscribe to immutable infrastructure best practices meaning essentially software infrastructure (hosts and kernels) remains unchanged as components are added and deleted from the network.         

- __Evolvability.__ Translates to innovation autonomy from the existing environment while not deviating from k8s-based deployment and operations best practicies.


It goes without saying that Pod Networking 2.0 must be 100% Kubernetes compliant as well as private, hybrid and public cloud friendly. In addition it is ideal to utitlize existing open source project software to build and deploy solutions. 

## Contivpp.io Solution       

Contivpp.io is a Kubernetes CNI network plugin designed and built to address pod networking 2.0 requirements. It uses [FD.io VPP](https://fd.io/)to provide policy and service-aware dataplane functionality consistent with k8s orchestration and lifecycle management.

 The important new features are:

- Maps k8s policy and service mechanisms (i.e. label selectors) to the FD.io/VPP dataplane. In effect, said policies and services are network-aware leading to the ability for containerized applications to benefit from optimal transport resources. 

- Implemented in user space for rapid innovation and immutability.

- Programmability enabled by an VPP agent that expedites the mapping of k8s policies and services into FD.io/VPP dataplance configuration.   


![contivpp.io Architecture](/img/what-is-contiv-vpp/contivpp-overview-pict4.png)

Contivpp.io is like any other CNI plugin and thus contains all of the usual "hooks" to boostrap and manage the network. Nothing new there. 

Conceptually at a high level, there are contivpp.io control plane components (ksr, etcd) on the master node that receive and store k8s policy and service updates in an etcd datastore. On all nodes, there are control plane components to listen for updates and programm the dataplane. The contiv-vswitch include the VPP agent and FD.io/VPP. 

The unique aspects of contivpp.io are:

* __k8s Policy and Service Mapping to FD.io/VPP Configuration.__ This is an automated distribution pipeline where k8s policies and services are automatically reflected into FD.io/VPP configuration information which is then programmed into the network.    

* __contiv-vswitch__ It is composed the FD.io/VPP dataplane and a Ligato-based VPP agent. It is across APIs enabled by the VPP agent that configruations rendered from policy and services are passed into the FD.io/VPP dataplane. The contiv-vswitch runs completely in userspace and uses [DPDK](https://dpdk.org/) for direct access to the network I/O layer. 



