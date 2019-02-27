+++
title = "Using Contiv-VPP Netctl"
type = "blog"
author = "Robin Tuszik"
date = "26 Feb 2019"
weight = "1"
+++ 

Contiv-VPP ```contiv-netctl``` is a command line interface (CLI) for querying information about Contiv-VPP components (include contiv-vswitches) operating inside a kubernetes (k8s) cluster.  This blopg covers contiv-netctl syntax, describes the available commands, and provides common examples.```contiv-netctl``` is installed on the K8s-master host during the installation of the Contiv-VPP CNI plugin.
<!--more--> 

In order to obtain information on nodes and pods, the Contiv-Netctl communicates with the Contiv-ETCD. The ETCD provides basic information including the status of contiv pods and the nodes where they reside. For information about contiv-vswitches and including VPP CLI support, contiv-netctl communicates with the Contiv-VPP Agent. There is one contiv-vpp agent per contiv-vswitch. This agent is connected to other Contiv-VPP parts of the Contiv setup. This is illustrated in the diagram below. 

![Contiv-netctl](/img/blog/contiv-netctl-arch.png)

*Contiv-VPP Architecture highlighting Contiv-Netctl*

<br/>

Note: `Contiv-netctl` is not to be confused with `kubectl`. Netctl is the CLI specifically for the Contiv-VPP deployment in a cluster, while kubectl is used for the entire Kubernetes cluster.

_To find out how to install Contiv-VPP using Vagrant [follow this link](https://github.com/contiv/vpp/blob/master/vagrant/README.md)._ 


# Accessing the Contiv-VPP Netctl

To access the contiv-netctl CLI, you must first access the master node.

After contiv-VPP is sucessfully deployed per the instructions above, follow these steps in order to access the k8s-master node:

cd into vpp/vagrant (`cd vpp/vagrant`)

From this directory log into the master node by using

```
Robin@MacbookPro:~/vpp/vagrant vagrant ssh k8s-master)
```

Now your CLI should look similar to this:

```
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
New release '18.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Tue Feb 26 10:56:26 2019 from 10.0.2.2
vagrant@k8s-master:~$
```

You can now use netctl by using the following syntax:
<br/>

```
vagrant@k8s-master:~$ contiv-netctl [operation] [node] [parameters] [flags]
```

where operation, node, parameters, and flags are:

* operation: specifies the query operation to be performed on one or more vswitches in the cluster


* node: optional, specifies the node where the query operation is to be executed. For some query operations, if node is not specified, the operation is performed on all nodes.


* parameters: optional, specifies parameters to the query operation


* flags: Specifies optional flags. Currently, only the -h flag is supported; when specified, help text for the specified operation(s) is printed to stdout.



The following commands are Possible: 
`help`,
`ipam`,
`nodes`,
`pods`,
`vppcli`,
`vppdump`

In the following section you will find a more detailed review of the commands you can use.
<br/>

## NETCTL COMMANDS

#### Node Information

The command `contiv-netctl nodes` will output vswitch information for all nodes running in the contiv cluster. The output will be something like this:

```
vagrant@k8s-master:~$ contiv-netctl nodes

ID  NODE-NAME    VPP-IP        HOST-IP     START-TIME                STATE  BUILD-VERSION  BUILD-DATE
1   k8s-master   192.168.16.1  10.20.0.2   Thu Feb 14 19:57:20 2019  OK     v2.1.1         Thu Jan 24 11:45:00 2019
2   k8s-worker1  192.168.16.2  10.20.0.10  Thu Feb 14 19:56:48 2019  OK     v2.1.1         Thu Jan 24 11:45:00 2019
3   k8s-worker2  192.168.16.3  10.20.0.11  Thu Feb 14 19:57:11 2019  OK     v2.1.1         Thu Jan 24 11:45:00 2019
```

* VPP-IP: IP-address of the host used for network connectivity as well as the tunnel end-point IP addresses for inter-pod vxlan tunnels.

* HOST-IP: The IP-address of the host the node lies on

* START-TIME: Shows you the time when the node was first started up

* STATE: The health of the contiv-vswitch in the node

* BUILD-VERSION: The version of the Contiv-agent that is running on the node

* BUILD-DATE: The date that build was released

***

> In order to find out more about the networking of Contiv-VPP refer [to this site](https://contivpp.io/docs/components/networking/)

***
<br/>

#### Pod Information

The command `contiv-netctl pods` will output network information of all the pods connected to the specified node. If the node has not been specified it will show the pod information for all nodes.

The output for this command will be similar to this:

```
vagrant@k8s-master:~$ contiv-netctl pods


k8s-master (10.20.0.2):
-----------------------
POD-NAME                               NAMESPACE    POD-IP      IF-IDX  IF-NAME
contiv-crd-nc9dw                       kube-system  10.20.0.2
contiv-etcd-0                          kube-system  10.20.0.2
contiv-ksr-sfh7t                       kube-system  10.20.0.2
contiv-ui-596557c887-kh7wk             kube-system  10.1.1.4    9       tap3
contiv-vswitch-c42xh                   kube-system  10.20.0.2
coredns-78fcdf6894-5zln4               kube-system  10.1.1.2    7       tap1
coredns-78fcdf6894-gz6mt               kube-system  10.1.1.3    8       tap2
etcd-k8s-master                        kube-system  10.20.0.2
kube-apiserver-k8s-master              kube-system  10.20.0.2
kube-controller-manager-k8s-master     kube-system  10.20.0.2
kube-proxy-c96g8                       kube-system  10.20.0.2
kube-scheduler-k8s-master              kube-system  10.20.0.2

k8s-worker1 (10.20.0.10):
-------------------------
POD-NAME                               NAMESPACE    POD-IP      IF-IDX  IF-NAME
contiv-vswitch-kpslq                   kube-system  10.20.0.10
kube-proxy-lrkrl                       kube-system  10.20.0.10

k8s-worker2 (10.20.0.11):
-------------------------
POD-NAME                               NAMESPACE    POD-IP      IF-IDX  IF-NAME
contiv-vswitch-7dgm4                   kube-system  10.20.0.11
kube-proxy-hswcn                       kube-system  10.20.0.11
kubernetes-dashboard-7f6874447b-95hq2  kube-system  10.1.3.2    7       tap1
```


<br/>

#### IPAM Information

IPAM stands for IP Address Management.
The command `contiv-netctl ipam` will show you ip address information for a specified node. If you do not specify the node, the information for all nodes will be shown.

The output of this command should look similar to this:

```
vagrant@k8s-master:~$ contiv-netctl ipam

ID  NODE-NAME    VPP-IP        BVI-IP        POD-CIDR     VPP-2-HOST-CIDR  POD-CLUSTER-CIDR
1   k8s-master   192.168.16.1  192.168.30.1  10.1.1.0/24  172.30.1.0/24    10.1.0.0/16
2   k8s-worker1  192.168.16.2  192.168.30.2  10.1.2.0/24  172.30.2.0/24    10.1.0.0/16
3   k8s-worker2  192.168.16.3  192.168.30.3  10.1.3.0/24  172.30.3.0/24    10.1.0.0/16
```

* VPP-IP: Shows the IP-Address of the pod on the contiv vswitch

* BVI-IP: The IP-Address of the Bride-Virtual-Interface

* POD-CIDR: The IP range the pods on this node will be in

* VPP-2-HOST-CIDR: The IP range that connects VPP to the host stack
	
* POD-CLUSTER-CIDR: The IP range the pod clusters will be in on the specified node


<br/>




#### Executing VPP debug CLI command

The command `contiv-netctl vppcli` will execute the specified VPP debug CLI command on the node that has been specified.

In order to find the list of possible CLI commands use the following command: `contiv-netctl vppcli k8s-master ?`

One useful VPPCLI Command is `contiv-netctl vppcli k8s-master sh version`
This command will give you the software version of the contiv vswitch.

```
vagrant@k8s-master:~$ contiv-netctl vppcli k8s-master sh version

"vpp v18.10-22~g13f5dcf9-dirty built by root on 4450257c4ced at Thu Jan 24 08:42:51 UTC 2019
"
```


Another example command is `contiv-netctl vppcli k8s-master sh int addr`
When using this command you should get something like this:

```
vagrant@k8s-master:~$ contiv-netctl vppcli k8s-master sh int addr

"GigabitEthernet0/8/0 (up):
  L3 192.168.16.1/24
local0 (dn):
loop0 (up):
  L3 10.1.1.1/24 ip4 table-id 1 fib-idx 1
loop1 (up):
  L2 bridge bd-id 1 idx 1 shg 1 bvi
  L3 192.168.30.1/24 ip4 table-id 1 fib-idx 1
tap0 (up):
  L3 172.30.1.1/24
tap1 (up):
  unnumbered, use loop0
  L3 10.1.1.1/24 ip4 table-id 1 fib-idx 1
tap2 (up):
  unnumbered, use loop0
  L3 10.1.1.1/24 ip4 table-id 1 fib-idx 1
tap3 (up):
  unnumbered, use loop0
  L3 10.1.1.1/24 ip4 table-id 1 fib-idx 1
vxlan_tunnel0 (up):
  L2 bridge bd-id 1 idx 1 shg 1
vxlan_tunnel1 (up):
  L2 bridge bd-id 1 idx 1 shg 1
"
```

#### VPPDump

The command `vppdump` allows you to execute the following commands that will help with basic debugging:

```
vagrant@k8s-master:~$ contiv-netctl vppdump

cmd 0: linux-interface-watcher
cmd 1: microservice
cmd 2: linux-interface
cmd 3: linux-arp
cmd 4: linux-route
cmd 5: vpp-acl-to-interface
cmd 6: vpp-bd-interface
cmd 7: vpp-interface
cmd 8: vpp-acl
cmd 9: vpp-arp
cmd 10: vpp-bridge-domain
cmd 11: vpp-dhcp
cmd 12: vpp-ip-scan-neighbor
cmd 13: vpp-l2-fib
cmd 14: vpp-nat44-dnat
cmd 15: vpp-nat44-global
cmd 16: vpp-nat44-interface
cmd 17: vpp-proxy-arp
cmd 18: vpp-proxy-arp-interface
cmd 19: vpp-punt-ip-redirect
cmd 20: vpp-punt-to-host
cmd 21: vpp-static-route
cmd 22: vpp-stn-rules
cmd 23: vpp-unnumbered-interface
cmd 24: vpp-xconnect
```


One of the possible commands is `contiv-netctl vppdump k8s-master linux-interface-watcher`


This is an example of an output for this command:

```
vagrant@k8s-master:~$ contiv-netctl vppdump k8s-master linux-interface-watcher

[
  {
    "Key": "linux/interface/host-name/lo",
    "Value": {},
    "Metadata": null,
    "Origin": 2
  },
  {
    "Key": "linux/interface/host-name/enp0s3",
    "Value": {},
    "Metadata": null,
    "Origin": 2
  },
  {
    "Key": "linux/interface/host-name/enp0s9",
    "Value": {},
    "Metadata": null,
    "Origin": 2
  },
  {
    "Key": "linux/interface/host-name/docker0",
    "Value": {},
    "Metadata": null,
    "Origin": 2
  },
  {
    "Key": "linux/interface/host-name/vpp1",
    "Value": {},
    "Metadata": null,
    "Origin": 2
  }
]

```
As outputs of this and other vppdump commands are provided in VPP "CLI-ese" or JSON formats, it is beneficial to possess in-depth knowledge of VPP, kubernetes clusters and how Contiv-VPP works. Because of this and a unwritten rule of confining blog posts to a maximum 5-minute read, I did not include detailed explanations of all outputs.

### About the Author

Robin is a German student who spent some time as an intern with the Contiv-VPP team at Cisco in San Jose. While casually using Contiv-VPP like any other normal person does, it one day struck him - There must be a CLI for Contiv-VPP! He began to dive deeper into this topic and the result was this blog.

You can reach Robin at robin@tuszik.de
