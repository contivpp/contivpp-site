+++
title = "Brief Description"
type = "concepts"
summary = "Outlines the problem statment and several important aspects of the Contiv - VPP Solution"
weight = "1"
+++



# Contiv-VPP Kubernetes Network Plugin


## Problem Statement

Cloud native interest and uptake is growing fast. With that comes the need for networking to keep pace, in particular where pod-to-pod and pod-to-service connectivity is concerned. Yes the notion of a Kubernetes CNI network plugin defining an API for managing cluster pod networks is there. And there are multiple tested and deployed [Container Network Interface (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) network plugins enabling pods on the same or different hosts to talk with each other. One could say that Kubernetes Pod Networking 1.0 is complete.

What is needed for K8s Pod Networking 2.0?

- __Policies and Services are made first-class citizens__ which can be restated to mean k8s policy and service awareness at the network layer.

- __Performance.__ Cloud native deployment expansion coupled with application workloads sensitive to network resources constraints is a reality and performance is mandatory. This includes any/all data-plane features to realize this performance objective including but not limited to such features as ACLs, NAT, IPsec and tunnel encap/decap.

- __Interface Versatility.__ Must accomodate your classic kernel interfaces (e.g. tap, veth) and newer, performance optimized interfaces such as memif.  

- __Observability.__ There are never enough pushed or pulled stats, telemetry, logs, tracing etc. data to be consumed by management, collection and analytics apps. 2.0 must include mechanisms to needed for detailed awareness and visibility of network behavior and performance.  

- __API Support.__ The k8s API is ideal for K8s cluster management but somewhat limited cluster networking. Existing CNI plugins incorprate their own API implementation resulting in potential lock-in dependences. API development tools, plugins and management agents makes it easy to build applications and services to interact with the underlying networking. It also allows cloud native and CNI developers to devote cycles to what is really important: applications.

- __Fast Configuration Programmability__. Pod/container setup and teardown is fast and furious. There is really no time to wait for configuration updates (i.e. IP addresses) from a central entity. It is faster and more accurate to "watch" a datastore for any changes and immediately execute related configuration updates. Likewise the configuration is there once a pod resurfaces post-crash.

- __CNF.__ Embrace cloud native network functions where appropriate. This means introducing CNFs into a k8s cluster dataplane and/or to construct a service chain. 

- __Immutablity.__ Must subscribe to immutable infrastructure best practices meaning essentially software infrastructure (hosts and kernels) remains unchanged as components are added and deleted from the network.         

- __Evolvability.__ Translates to innovation autonomy from the existing environment while not deviating from k8s-based deployment and operations best practicies.


It goes without saying that Pod Networking 2.0 must be 100% Kubernetes compliant and private, hybrid and public cloud-friendly. In addition it is ideal to utilize existing open source project software to build and deploy solutions. 

# Contiv-VPP Kubernetes Network Plugin

##  Solution       

 Contiv-VPP is a Kubernetes CNI network plugin designed and built to address pod networking 2.0 requirements. It uses [FD.io VPP](https://fd.io/) to provide policy and service-aware dataplane functionality consistent with k8s orchestration and lifecycle management.

 The important features are:

- Maps k8s policy and service mechanisms (i.e. label selectors) to the FD.io/VPP dataplane. In effect, said policies and services are network-aware leading to the ability for containerized applications to benefit from optimal transport resources. 

- Implemented in user space for rapid innovation and immutability.

- Programmability enabled by a [Ligato-based VPP agent](https://ligato.io) that expedites the mapping of k8s policies and services into a FD.io/VPP dataplane configuration.   


![ Architecture](/img/what-is-contiv-vpp/contivpp-overview-pict4.png)

Contiv-VPP is like any other CNI plugin and thus contains all of the usual "hooks" to bootstrap and manage the network. Nothing new there. 

Conceptually at a high level, there are Contiv-VPP control plane components (ksr, etcd) on the master node that receive and store k8s policy and service updates in an etcd datastore. On all nodes, there are contiv components that listen for updates and program the dataplane. The contiv-vswitch includes the Ligato VPP agent and FD.io/VPP packaged in a single container.

The unique aspects of Contiv-VPP are:

* __k8s Policy and Service Mapping to FD.io/VPP Configuration.__ This is an automated distribution pipeline where k8s policies and services are automatically reflected into FD.io/VPP configuration information which is then programmed into the network.    

* __contiv-vswitch__ It is composed the FD.io/VPP dataplane and the Ligato VPP agent. It is across APIs enabled by the Ligato VPP agent/plugins that configurations rendered from policy and services are pushed down into the FD.io/VPP dataplane. The contiv-vswitch runs completely in userspace and uses [DPDK](https://dpdk.org/) for direct access to the network I/O layer. 

* __IPv6/SRv6__ In many scenarios, a k8s requires the use of a network address translation (NAT) component so client pods can access services identified with an internal clusterIP address. This introduces another level of indirection which adds complexity and could impact scale/performance. IPv6 CNI configuration is now supported. In addition, segment routing IPv6 which is a form of source-routing using IPv6 addresses is now implemented. No NAT and IPv6 support are the value-adds.



