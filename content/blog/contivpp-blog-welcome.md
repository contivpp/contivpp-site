+++
title = "Contiv VPP Demo APIs and Commands"
date = 2018-08-17T16:18:07-05:00
summary = "Lots coming on a raft of bloggy-things, contivpp.io and otherwise over the next 30 days and beyond. Check 'em Out!"
tags = ["contivpp.io"]
categories = ["contivpp.io"]
draft = true
weight = 1
+++


Use the [Contiv-VPP Vagrant Demo](https://github.com/contiv/vpp/blob/master/vagrant/README.md) to experiment and play with the following APIs and commands. 

<!--more-->
## contiv-netctl Commands

### What's possible

	vagrant@k8s-master:~$ contiv-netctl -h
	Usage:
	  netctl [command]

	Available Commands:
	  help        Help about any command
	  ipam        Shows IPAM information for specified node.
	  nodes       Shows vswitch information for all nodes in the running Contiv cluster.
	  pods        Display network information for pods connected to VPP on the given node. If node is omitted, pod data for all nodes is shown.
	  vppcli      Execute the specified VPP debug CLI command on the specified node.
	  vppdump     Print anything to the screen

	Flags:
	  -h, --help   help for netctl

	Use "netctl [command] --help" for more information about a command.

###Nodes Example

	vagrant@k8s-master:~$ contiv-netctl nodes

	ID  NODE-NAME    VPP-IP        HOST-IP     START-TIME                STATE  BUILD-VERSION  BUILD-DATE
	1   k8s-master   192.168.16.1  10.20.0.2   Wed Oct 17 13:49:54 2018  1      v1.4.1         Wed Oct 10 13:21:00 2018
	2   k8s-worker1  192.168.16.2  10.20.0.10  Wed Oct 17 13:50:23 2018  1      v1.4.1         Wed Oct 10 13:21:00 2018
	3   k8s-worker2  192.168.16.3  10.20.0.11  Wed Oct 17 13:51:26 2018  1      v1.4.1         Wed Oct 10 13:21:00 2018



###Pods Example

	vagrant@k8s-master:~$ contiv-netctl pods

	k8s-master:
	-----------
	POD-NAME                  NAMESPACE    POD-IP     VPP-IP     IF-IDX  IF-NAME  INTERNAL-IF-NAME    HOST-IP
	coredns-78fcdf6894-sh6dk  kube-system  10.1.1.10  10.2.1.10  8       tap3     tape36dab58a8c6f66  10.20.0.2
	coredns-78fcdf6894-xwccm  kube-system  10.1.1.11  10.2.1.11  3       tap1     tap8b19659544a1045  10.20.0.2


	k8s-worker1:
	------------
	POD-NAME                  NAMESPACE    POD-IP     VPP-IP     IF-IDX  IF-NAME  INTERNAL-IF-NAME    HOST-IP
	nginx-64f497f8fd-t72kn    default      10.1.2.4   10.2.1.4   7       tap2     tapf4ac6f91799eaae  10.20.0.10



	k8s-worker2:
	------------
	POD-NAME                  NAMESPACE    POD-IP     VPP-IP     IF-IDX  IF-NAME  INTERNAL-IF-NAME    HOST-IP
	nginx-64f497f8fd-rlkg7    default      10.1.3.4   10.2.1.4   7       tap2     tapa8e72df69069a93  10.20.0.11



###IPAM Example

	vagrant@k8s-master:~$ contiv-netctl ipam

	ID  NODE-NAME    VPP-IP        BVI-IP        POD-NET-CIDR  VPP-2-HOST-CIDR  POD-IFIP-CIDR  POD-SUBNET-CIDR
	1   k8s-master   192.168.16.1  192.168.30.1  10.1.1.0/24   172.30.1.0/24    10.2.1.0/24    10.1.0.0/16
	2   k8s-worker1  192.168.16.2  192.168.30.2  10.1.2.0/24   172.30.2.0/24    10.2.1.0/24    10.1.0.0/16
	3   k8s-worker2  192.168.16.3  192.168.30.3  10.1.3.0/24   172.30.3.0/24    10.2.1.0/24    10.1.0.0/16



###vppcli example

	vagrant@k8s-master:~$ contiv-netctl vppcli k8s-worker1 sh int addr
	vppcli k8s-worker1 sh int addr
	GigabitEthernet0/8/0 (up):
	  L3 192.168.16.2/24
	local0 (dn):
	loop0 (up):
	  L2 bridge bd-id 1 idx 1 shg 1 bvi
	  L3 192.168.30.2/24 ip4 table-id 1 fib-idx 1
	tap0 (up):
	  L3 172.30.2.1/24
	tap2 (up):
	  L3 10.2.1.4/32 ip4 table-id 1 fib-idx 1
	vxlan_tunnel0 (up):
	  L2 bridge bd-id 1 idx 1 shg 1
	vxlan_tunnel1 (up):
	  L2 bridge bd-id 1 idx 1 shg 1





## Contiv VPP REST APIs

###IPAM Example

	vagrant@k8s-master:~$ curl http://localhost:9999/contiv/v1/ipam
	{
	  "nodeId": 1,
	  "nodeName": "k8s-master",
	  "nodeIP": "192.168.16.1",
	  "podNetwork": "10.1.1.0/24",
	  "vppHostNetwork": "172.30.1.0/24",
	  "config": {
	    "podIfIPCIDR": "10.2.1.0/24",
	    "podSubnetCIRDR": "10.1.0.0/16",
	    "podNetworkPrefixLen": 24,
	    "vppHostSubnetCIDR": "172.30.0.0/16",
	    "vppHostNetworkPrefixLen": 24,
	    "nodeInterconnectCIDR": "192.168.16.0/24",
	    "nodeInterconnectDHCP": false,
	    "vxlanCIDR": "192.168.30.0/24",
	    "serviceCIDR": "10.96.0.0/12"
	  }
	}

###VPP Agent REST APIs

[REST API Plugin Doc](https://github.com/ligato/vpp-agent/tree/master/plugins/rest)

###To Retrieve API Index

	vagrant@k8s-master:~$curl http://0.0.0.0:9999/

### Here is the API Index

Note: API Index REST call was executed from the local terminal of the [Contiv-VPP Vagrant Demo](https://github.com/contiv/vpp/blob/master/vagrant/README.md). To run one of these APIs in that environment, prepend `curl http://0.0.0.0:9999/` to one of the API index entries.

#### ACL plugin:

*   [IP-type access lists](/vpp/dump/v1/acl/ip)
*   [MACIP-type access lists](/vpp/dump/v1/acl/macip)

#### IPSec plugin:

*   [Security policy databases](/vpp/dump/v1/ipsec/spd)
*   [Security associations](/vpp/dump/v1/ipsec/sa)
*   [Tunnel interfaces](/vpp/dump/v1/ipsec/tunnel)

#### Interface plugin:

*   [All interfaces](/vpp/dump/v1/interfaces)
*   [Loopbacks](/vpp/dump/v1/interfaces/loopback)
*   [Ethernets](/vpp/dump/v1/interfaces/ethernet)
*   [Memifs](/vpp/dump/v1/interfaces/memif)
*   [Taps](/vpp/dump/v1/interfaces/tap)
*   [VxLANs](/vpp/dump/v1/interfaces/vxlan)
*   [Af-packets](/vpp/dump/v1/interfaces/afpacket)

#### L2 plugin:

*   [Bridge domains](/vpp/dump/v1/bd)
*   [Bridge domain IDs](/vpp/dump/v1/bdid)
*   [L2Fibs](/vpp/dump/v1/fib)
*   [Cross connects](/vpp/dump/v1/xc)

### L3 plugin:

*   [Routes](/vpp/dump/v1/routes)
*   [ARPs](/vpp/dump/v1/arps)
*   [Proxy ARP interfaces](/vpp/dump/v1/proxyarp/interfaces)
*   [Proxy ARP ranges](/vpp/dump/v1/proxyarp/ranges)

#### L4 plugin:

*   [L4 sessions](/vpp/dump/v1/sessions)

#### Telemetry:

*   [All data](/vpp/telemetry)
*   [Memory](/vpp/telemetry/memory)
*   [Runtime](/vpp/telemetry/runtime)
*   [Node count](/vpp/telemetry/nodecount)

#### Tracer:

*   [Binary API](/vpp/binapitrace)


## Some Kubernetes APIs

[Kubernetes API Access Cluster Doc](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)



### Examples

	vagrant@k8s-master:~$ curl http://localhost:8080/api/v1/pods

	vagrant@k8s-master:~$ curl http://localhost:8080/api/v1/nodes

	vagrant@k8s-master:~$ curl http://localhost:8080/api/v1/services



