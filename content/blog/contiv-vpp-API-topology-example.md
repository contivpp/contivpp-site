+++
title = "Using K8s and Contiv-VPP APIs to Discover a Cluster Network Topology"
type = "blog"
author = "Chris Metz"
date = "14 Nov 2018"
weight = "2"
+++ 

We know about Kubernetes (K8s) API. Contiv-VPP comes with its own set of REST APIs out of the box. You can exercise many of these APIs to gain a better understanding of how it works. In this blog, we'll look at how a set of K8s and Contiv VPP REST APIs can be used to discover a cluster network topology.  
<!--more-->

# Example Using K8s and Contiv-VPP APIs to Discover a Cluster Network Topology

We know about Kubernetes (K8s) API. Contiv-VPP comes with its own set of REST APIs out of the box. You can exercise many of these APIs to gain a better understanding of how it works. In this blog, we'll look at how a set of K8s and Contiv VPP REST APIs can be used to discover a cluster network topology.  This example covers a subset of the network but can be can easily replicated to construct the entire network topology.


![Here is a cluster using contiv-vpp and several application pods applied](/img/blog/1.png)Let's see what a subset of the topology appears from the viewpoint of k8s-worker2 in our 3 x node cluster. The illustrations used in this blog are lifted from our [Contiv VPP demo app](/demo/demo).

## VPP-IP Addresses 

What is a VPP-IP address?

From [github networking notes](https://github.com/contiv/vpp/blob/master/docs/NETWORKING.md):


	PodIfIPCIDR (default 10.2.1.0/24): VPP-internal addresses used to put the VPP interfaces facing towards 
	the PODs into L3 mode. This IP range will be reused on each node, thereby it is never externally 
	addressable outside of the node itself. The only requirement is that this subnet should not collide with any other IPAM subnet.


So, VPP-IP addresses facing pods are always 10.2.1.x/32 on any node. We need this because the other end of an interface (i.e. tap) is the IP address of a pod.


## Finding the POD-IP address

How do we discover the IP address of the pod? This contiv-netctl command provides the answer. 

	vagrant@k8s-master:~$contiv-netctl pods

	k8s-master:
	-----------
	POD-NAME                  NAMESPACE    POD-IP     VPP-IP     IF-IDX  IF-NAME  INTERNAL-IF-NAME    HOST-IP
	coredns-78fcdf6894-sh6dk  kube-system  10.1.1.20  10.2.1.20  8       tap3     tap05b94a5c0234e3d  10.20.0.2
	coredns-78fcdf6894-xwccm  kube-system  10.1.1.21  10.2.1.21  4       tap2     tap5556a0bd8a5178c  10.20.0.2

	k8s-worker1:
	------------
	POD-NAME                  NAMESPACE    POD-IP     VPP-IP     IF-IDX  IF-NAME  INTERNAL-IF-NAME    HOST-IP
	nginx-64f497f8fd-t72kn    default      10.1.2.10  10.2.1.10  7       tap2     tapb958777d69aea8d  10.20.0.10

	k8s-worker2:
	------------
	POD-NAME                  NAMESPACE    POD-IP     VPP-IP     IF-IDX  IF-NAME  INTERNAL-IF-NAME    HOST-IP
	nginx-64f497f8fd-rlkg7    default      `10.1.3.9`   10.2.1.9   7       tap2     tap5d62d2bb6c4774d  10.20.0.11

Here is how it appears using the contiv-netctl command modal in our app.


![Here is how it appears using the contiv-netctl command modal in our app](/img/blog/9.png)

We would like to use REST APIs to solve this problem. Not CLI solutions such as kubetctl or contiv-netctl. Let's continue. 


## Obtain the POD-IP Address Facing the contiv vswitch


The basic api/v1/pods provides a start.

	curl http://localhost:8080/api/v1/pods 

*Note* - Dont forget to enable the [K8s API Access functionality](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/) 


Look for those pods with a POD-IP that *DOES NOT* match the hostIP. Yes, that's right. You will also see this by examining the contiv-netctl pods command output shown above.


In this example we know there is an NGINX pod on k8s-worker2. Here is a snippet of the API call to that specific NGINX pod put together by taking k8s REST API call prefix.


	curl http://localhost:8080/


and prepending it to the selflink metadata contained in pod API call response metadata.

	/api/v1/namespaces/default/pods/nginx-64f497f8fd-rlkg7

	curl http://localhost:8080/api/v1/namespaces/default/pods/nginx-64f497f8fd-rlkg7

We use a snippet because the JSON response for all of the pods is massive.

In the response we have:

	"hostIP": "10.20.0.11",
    "podIP": "10.1.3.9",


NGINX pod IP address is 10.1.3.9 running on the K8s-worker2 host with an IP address of 10.20.0.11.


## Obtain the VPP-IP Address Facing the Pod


Next, we need to figure out VPP-IP interface and address. Arp tables contain the local IP and Mac addresses of hosts on the subnet. Therefore we can assume that ARP table on the contiv vswitch will contain the IP address of the Pod at the other end of the interface.

The ARP table maintained by VPP on k8s-worker2 can be retrieved using:


	vagrant@k8s-worker2:~$ curl http://0.0.0.0:9999/vpp/dump/v1/arps
	[
	  {
	    "Arp": {
	      "interface": "vxlanBVI",
	      "ip_address": "192.168.30.1",
	      "phys_address": "1a:2b:3c:4d:5e:01",
	      "static": true
	    },
	    "Meta": {
	      "SwIfIndex": 4
	    }
	  },
	  {
	    "Arp": {
	      "interface": "vxlanBVI",
	      "ip_address": "192.168.30.2",
	      "phys_address": "1a:2b:3c:4d:5e:02",
	      "static": true
	    },
	    "Meta": {
	      "SwIfIndex": 4
	    }
	  },
	  {
	    "Arp": {
	      "interface": "tap-vpp2",
	      "ip_address": "172.30.3.2",
	      "phys_address": "62:e8:96:e8:df:89"
	    },
	    "Meta": {
	      "SwIfIndex": 2
	    }
	  },
	  {
	    "Arp": {
	      "interface": "tap5d62d2bb6c4774d",
	      "ip_address": "10.1.3.9",
	      "phys_address": "00:00:00:00:00:02",
	      "static": true
	    },
	    "Meta": {
	      "SwIfIndex": 7
	    }
	  }
	]


Looking at the response, we see that the interface with "SwIfIndex": 7 has an ARP cache entry for 10.1.3.9. Our NGINX podIP! There is an interface on the contiv vswitch connecting the NGINX POD-IP address of 10.1.3.9.


Next, we need to find out the IP address of this interface. This can be accomplished by calling the interfaces REST API on K8s-worker2 and use "SwIfIndex": 7 to look up the IP address (for brevity sake, we only retrieve tap interfaces).


	vagrant@k8s-worker2:~$ curl http://0.0.0.0:9999/vpp/dump/v1/interfaces/tap
	{
	  "2": {
	    "interface": {
	      "name": "tap-vpp2",
	      "type": 3,
	      "enabled": true,
	      "phys_address": "01:23:45:67:89:42",
	      "mtu": 1450,
	      "ip_addresses": [
	        "172.30.3.1/24"
	      ],
	      "tap": {
	        "version": 2
	      }
	    },
	    "interface_meta": {
	      "sw_if_index": 2,
	      "tag": "tap-vpp2",
	      "internal_name": "tap0",
	      "dhcp": null
	    }
	  },
	  "7": {
	    "interface": {
	      "name": "tap5d62d2bb6c4774d",
	      "type": 3,
	      "enabled": true,
	      "phys_address": "02:fe:fc:07:21:82",
	      "mtu": 1450,
	      "vrf": 1,
	      "ip_addresses": [
	        "10.2.1.9/32"
	      ],
	      "tap": {
	        "version": 2
	      }
	    },
	    "interface_meta": {
	      "sw_if_index": 7,
	      "tag": "tap5d62d2bb6c4774d",
	      "internal_name": "tap2",
	      "dhcp": null
	    }
	  }
	}


And we see that "SwIfIndex": 7 has an IP address of 10.2.1.9. This is the VPP-IP address of the interface (L3) facing the NGINX pod with a POD-IP of 10.1.3.9.

NGINX POD-IP = 10.1.3.9; VPP-IP = 10.2.1.9. We used REST APIs to discover the IP addresses.



## Show Node Interfaces

As shown above, a network topology should render all interfaces and associated information on all nodes.

Here is the REST API on k8s-master to retrieve all interfaces on this node. It can performed on the other nodes as well.


	vagrant@k8s-master:~$ curl http://0.0.0.0:9999/vpp/dump/v1/interfaces
	{
	  "0": {
	    "interface": {},
	    "interface_meta": {
	      "sw_if_index": 0,
	      "tag": "",
	      "internal_name": "local0",
	      "dhcp": null
	    }
	  },
	  "1": {
	    "interface": {
	      "name": "GigabitEthernet0/8/0",
	      "type": 1,
	      "enabled": true,
	      "phys_address": "08:00:27:62:d2:5e",
	      "mtu": 9202,
	      "ip_addresses": [
	        "192.168.16.1/24"
	      ]
	    },
	    "interface_meta": {
	      "sw_if_index": 1,
	      "tag": "GigabitEthernet0/8/0",
	      "internal_name": "GigabitEthernet0/8/0",
	      "dhcp": null
	    }
	  },
	  "2": {
	    "interface": {
	      "name": "tap-vpp2",
	      "type": 3,
	      "enabled": true,
	      "phys_address": "01:23:45:67:89:42",
	      "mtu": 1450,
	      "ip_addresses": [
	        "172.30.1.1/24"
	      ],
	      "tap": {
	        "version": 2
	      }
	    },
	    "interface_meta": {
	      "sw_if_index": 2,
	      "tag": "tap-vpp2",
	      "internal_name": "tap0",
	      "dhcp": null
	    }
	  },
	  "3": {
	    "interface": {
	      "name": "tapcf4729237687888",
	      "type": 3,
	      "enabled": true,
	      "phys_address": "02:fe:16:3f:5f:0f",
	      "mtu": 1450,
	      "vrf": 1,
	      "ip_addresses": [
	        "10.2.1.23/32"
	      ],
	      "tap": {
	        "version": 2
	      }
	    },
	    "interface_meta": {
	      "sw_if_index": 3,
	      "tag": "tapcf4729237687888",
	      "internal_name": "tap1",
	      "dhcp": null
	    }
	  },
	  "5": {
	    "interface": {
	      "name": "vxlanBVI",
	      "enabled": true,
	      "phys_address": "1a:2b:3c:4d:5e:01",
	      "vrf": 1,
	      "ip_addresses": [
	        "192.168.30.1/24"
	      ]
	    },
	    "interface_meta": {
	      "sw_if_index": 5,
	      "tag": "vxlanBVI",
	      "internal_name": "loop0",
	      "dhcp": null
	    }
	  },
	  "6": {
	    "interface": {
	      "name": "vxlan2",
	      "type": 5,
	      "enabled": true,
	      "vxlan": {
	        "src_address": "192.168.16.1",
	        "dst_address": "192.168.16.2",
	        "vni": 10
	      }
	    },
	    "interface_meta": {
	      "sw_if_index": 6,
	      "tag": "vxlan2",
	      "internal_name": "vxlan_tunnel0",
	      "dhcp": null
	    }
	  },
	  "7": {
	    "interface": {
	      "name": "vxlan3",
	      "type": 5,
	      "enabled": true,
	      "vxlan": {
	        "src_address": "192.168.16.1",
	        "dst_address": "192.168.16.3",
	        "vni": 10
	      }
	    },
	    "interface_meta": {
	      "sw_if_index": 7,
	      "tag": "vxlan3",
	      "internal_name": "vxlan_tunnel1",
	      "dhcp": null
	    }
	  },
	  "8": {
	    "interface": {
	      "name": "tap44d1ba38360bf71",
	      "type": 3,
	      "enabled": true,
	      "phys_address": "02:fe:fc:07:21:82",
	      "mtu": 1450,
	      "vrf": 1,
	      "ip_addresses": [
	        "10.2.1.22/32"
	      ],
	      "tap": {
	        "version": 2
	      }
	    },
	    "interface_meta": {
	      "sw_if_index": 8,
	      "tag": "tap44d1ba38360bf71",
	      "internal_name": "tap3",
	      "dhcp": null
	    }
	  }
	}



We need to filter by vrf and type.



## Show Pod Networking Topology (or part of it)

Here we want to show how contiv-vpp sets up the network so that pods on different hosts (nodes) can talk to each other. Logically speaking,
we have pods in a VRF connected up with a vxlan bridge virtual interface (vxlanBVI) inside a host. The vxlanBVI interface and associated vxlan tunnel interfaces make up the bridge domain.


- previous section showed us how to learn which pods are connected to which interface (usually tap) on the vswitch. Again, inside a single node.


- REST API interfaces call (shown above) returns all interfaces on the vswitch inside a host. Tap interfaces with VPP-IP addresses (10.2.1.x) and the vxlanBVI interfaces are contained in a VRF. So a VRF interface set is (tap1, tapn, ... vxlanBVI). Note that vxlan tunnel interfaces along with their source and destination IP addresses are included in the API response as well.


Next we use the REST API bridge domain call to return the bridge domain and associated interfaces. 

	vagrant@k8s-master:~$ curl http://0.0.0.0:9999/vpp/dump/v1/bd
	{
	  "1": {
	    "bridge_domain": {
	      "name": "vxlanBD",
	      "forward": true,
	      "interfaces": [
	        {
	          "name": "vxlanBVI",
	          "bridged_virtual_interface": true,
	          "split_horizon_group": 1
	        },
	        {
	          "name": "vxlan2",
	          "split_horizon_group": 1
	        },
	        {
	          "name": "vxlan3",
	          "split_horizon_group": 1
	        }
	      ]
	    },
	    "bridge_domain_meta": {
	      "bridge_domain_id": 1,
	      "bridge_domain_id_to_name": {
	        "5": "vxlanBVI",
	        "6": "vxlan2",
	        "7": "vxlan3"
	      }
	    }
	  }
	}


We see vxlanBVI and the two vxlan tunnel interfaces (e.g. this is k8-master so one tunnel to worker1 and another to worker2). One note about IPAM addressing for vxlanBVI and vxlan tunnels:

- vxlanBVI IP addresses come from: vxlanCIDR: 192.168.30.0/24.

- vxlan interface tunnel source and destination IP addresses come from: nodeInterconnectCIDR: 192.168.16.0/24. Note vxlan tunnel IP addresses on nodes are terminated on the physical interface (e.g. GigabitEthernet0/8/0).

So there you have. This involved a bit of "restful" gymnastics but we were able to see under the covers how K8s and contiv-vpp employ REST APIs to retrieve information from the network.

Start RESTing!
