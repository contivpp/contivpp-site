+++
title = "Description"
type = "concepts"
summary = "Outlines the problem statement and important aspects of the Contiv - VPP Solution"
weight = "1"
+++



# Contiv-VPP Kubernetes Network Plugin


## Problem Statement

Cloud native interest and uptake is growing fast. With that comes the need for networking to keep pace, in particular where performance, scale and service-awareness are concerned.

You have a  multitude of [Kubernetes CNI plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) to choose from, all providing you with basic pod network connectivity. You could say that Kubernetes pod networking 1.0 is complete.

<br>

![k8s-net](/img/what-is-contiv-vpp/k8s-net-one-two.drawio.svg)
<p style="text-align: center; font-weight: bold">Figure: K8s pod networking 1.0 evolving towards K8s pod networking 2.0</p>

<br>

What is needed for K8s pod networking 2.0?

- __Make policies and services first-class citizens.__ In effect, you have K8s policy and service awareness at the network layer.

- __Performance.__ Application workloads sensitive to network resource constraints is a reality. This includes applications using any/all data plane features such as ACLs, NAT, IPsec and tunnel encap/decap.

- __Interface Versatility.__ Must accommodate your classic kernel interfaces (e.g. tapv2, veth) and newer, performance optimized interfaces such as memif.  

- __Observability.__ You can never have enough pushed or pulled stats, telemetry, logs and tracing consumed by your management and observability apps. Any solution must include mechanisms providing you with detailed awareness and visibility of network behavior and performance.  

- __API Support.__ Existing CNI plugins incorporate their own API implementation resulting in potential lock-in dependencies. Open and documented API development tools, plugins and management agents makes it easy for you to build applications and services that leverage a high-performance network data plane. Now you can have your developer community devote cycles to what is really important: applications.

- __Fast Configuration Programmability__. Container setup and teardown is fast and furious. You don't have time to wait on configuration updates from a central repository. It is faster and more accurate to "watch" a data store for any changes and immediately execute the desired configuration updates. 

- __CNF.__ Embrace cloud native network functions where appropriate. This means introducing CNFs into a K8s cluster data plane and/or to construct a service chain. 

- __Immutablity.__ Subscribing to immutable infrastructure best practices enables your host and kernel infrastructure to remain unchanged as you add and delete resources from the network.         

- __Evolvability.__ Translates to innovation autonomy from the existing environment while not deviating from K8s-based deployment and operations best practices.


It goes without saying that pod networking 2.0 must be 100% Kubernetes compliant, and private/hybrid/public cloud-friendly.   


##  Solution       

 Contiv-VPP is a Kubernetes CNI network plugin designed and built to address pod networking 2.0 requirements. It uses [FD.io VPP](https://fd.io/) to provide policy and service-aware data plane functionality consistent with K8s orchestration and lifecycle management.

 Contiv-VPP features consist of the following:

- Maps K8s policy and service mechanisms such as label selectors to the FD.io/VPP data plane. Policies and services become network-aware enabling your containerized applications to receive optimal transport resources. 

- Implemented in user space for rapid innovation and immutability.

- Programmability enabled by a [Ligato-based VPP agent](https://ligato.io) that provides you with APIs for FD.io/VPP data plane configuration.   

- contiv-netctl for CLI management and control.

<br>

![ Architecture](/img/what-is-contiv-vpp/k8s-contiv-vpp-arch.png)
<p style="text-align: center; font-weight: bold">Figure: High-level overview of a cluster running Contiv-VPP</p>

<br>

Contiv-VPP is like any other CNI plugin that contains all of the usual "hooks" to bootstrap and manage the network. 

At a high level, you have Contiv-VPP components on the control plane node that receive and store K8s policy and service information in an etcd data store. Contiv-VPP node agents listen for updates and program the data plane accordingly. The contiv-vswitch includes the Ligato VPP agent and FD.io/VPP packaged in a single container.

The unique aspects of Contiv-VPP are:

* __K8s policy and service mapping to the FD.io/VPP configuration.__ You have an automated distribution pipeline that maps K8s policies and services to FD.io/VPP configuration information programmed into the network.    

* __contiv-vswitch__ composed of the FD.io/VPP data plane and the Ligato VPP agent. The contiv-vswitch runs in user space and uses [DPDK](https://dpdk.org/) for direct access to the network I/O layer. 

* __IPv6/SRv6__ In many scenarios, a K8s deployment requires a network address translation (NAT) component so that client pods can access services identified with an internal clusterIP address. This introduces another level of indirection that adds complexity and impacts scale and performance. Contiv-VPP supports IPv6/SRv6 CNI so you do not require a NAT function in your deployment. 



