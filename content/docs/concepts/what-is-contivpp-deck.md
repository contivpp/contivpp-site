+++
title = "Presentation"
type = "concepts"
summary = "Baseline Concepts, Architecture, Network Overlay, K8s Policies & Services, APIs/Contiv-Netctl Commands"
weight = "2"
+++



![contivpp.io Architecture](/img/what-is-contiv-vpp/slides/Slide01.png)

## Topics Discussed

* Baseline Concepts

* Architecture

* Network Overlay

* K8s Services and Policies

* APIs/Contiv-Netctl Commands  


<br/>
<br/>
<br/>
<br/>
![contivpp slide](/img/what-is-contiv-vpp/slides/Slide02.png)

## This is Contivpp.io in a nutshell

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide03.png)

<br/>
<br/>
<br/>
<br/>


![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide04.png)

## One form of K8s cluster networking involves interconnecting pods

* [4 x patterns of K8s Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

* CNI Network Plugins help "bootstrap" a cluster IP network at pod creation] 

* [Multiple CNI plugins](https://github.com/containernetworking/cni) to choose from. All provide pod connectivity and most come with function add-ons. Choice of which to use depends on many factors including but not limited desired function, service/policy handling, performance, security, operations and so on. 

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide05.png)

## All packets travel through the kernel

* Traditionally applications have relied on the kernel's mature TCP/IP stack.

* Perfectly fine for dishing out web server traffic.

* Evolving high performance apps could encounter a performance bottleneck with kernel networking

* Another issues to consider is what happens if the kernel network stack breaks (caused by mis-behaving app or something else)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide06.png)

## User Space Networking

* The app and accompanying network stack (e.g. TCP/IP, tunnel encap/decap, IPsec) reside in user space. It could now be referred to as a Cloud Native Network Function (CNF). See next slide.

Other advantages include:

* __Elevated performance throughput__ as now the user space CNF can talk directly to the physical network interface card (NIC) with the goal of keeping pace with the speed developments happening in this space.

* __Accelerated network innovation__ development and roll-out. CNF developers can go to town and paint their innovations on a large user space canvas. It is THE opportunity to mandate all CNFs run in user space. It just make sense.

* __Fast recovery__. If anything happens to the user space CNF stack (e.g. upgrade, crash, etc.), it DOES NOT bring down the whole node. You just restart it quickly and continue on with your work.

High performance shared memory communications when talking user space - to - user space. Memif for inter-VPP traffic is one example.

<br/>
<br/>
<br/>
<br/>


![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide07.png)

## Some definitions of a CNF

* For now, let's assume a CNF provides L2, L3 and is L4-aware for the purposes of NAT and ACL.

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide08.png)

## CNF Software Dataplane

* Really could be for ANY network and/or application function

* [FD.io](http://fd.io)

* [FD.io/VPP Whitepaper](https://fd.io/wp-content/uploads/sites/34/2017/07/FDioVPPwhitepaperJuly2017.pdf)


<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide09.png)

## Ligato

[Ligato](https://ligato.io)

Ligato is an open source project that provides a platform and code samples for development of cloud native VNFs. 
It includes a management/control agent for VPP and a Service Function Chain (SFC) Controller for stitching virtual and physical networking.

<br/>
<br/>
<br/>
<br/>
![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide10.png)

## Baseline CNF vSwitch

[CNFs with a Dose of Ligato and FD.io/VPP](https://contivpp.io/blog/cnf-ligato-fdio/)



<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide11.png)


<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide12.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide13.png)

## Kubernetes State Reflector

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide14.png)

## Contiv ETCD

* Specific to Contivpp. Need to avoid any latency associated with system ETCD.


<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide15.png)

## Contiv vSwitch



<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide16.png)

## Contiv VPP Agent

* Based on the Ligato VPP agent with some modifications implemented for contivpp.

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide17.png)

## CNI amd STN


<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide18.png)

## Topolopgy Graph of a 3 x Node Contivpp Network

* contiv vSwitches are contained in each node including the master


<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide19.png)


<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide20.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide21.png)

## IPAM

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide22.png)

## Example of contiv vSwitch configuration with addressing provided by IPAM
<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide23.png)

## Logical View of a Contivpp Network Overlay





<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/contivpp-network-overlay-nodes.png)
## Depicts a 3 x node Contivpp Tunnel Overlay

* 5 x NGINX pods are networked together 

* vxlan tunnel encap is used but other possibilities include SRv6 and MPLS UDP
<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/contivpp-network-overlay-pods.png)
## Application Pods 

* set of pods connected to the overlay
<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/contivpp-network-overlay-tunnels.png)

## Pods on each node are connected by vxlan tunnels

* one in each direction between each set of nodes

* forms a full mesh with a unique virtual network interface (VNI)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide24.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide25.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide26.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide27.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide28.png)

## Contivpp K8s Service VPP-State

* on the left are the service endpoints are their IP addresses

* on the right is a snippet of REST API request returning the VPP-NAT entries in the contiv vSwitch

* this enables K8s service traffic to benefits from the VPP-based high performance dataplane 

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide29.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide30.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide31.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide32.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide33.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide34.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide35.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide36.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide37.png)

<br/>
<br/>
<br/>
<br/>



![contivpp.io slide](/img/what-is-contiv-vpp/slides/contivpp-contiv-netctl.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide38.png)

<br/>
<br/>
<br/>
<br/>

![contivpp.io slide](/img/what-is-contiv-vpp/slides/Slide39.png)



