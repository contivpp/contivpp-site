+++
title = "Packetflow"
type = "components"
weight = "4"
+++
Based on K8s policies and services along with reachability requirements, Contivpp will "wire up" the network. Once done, there will be a number of different paths traversed by payloads between pods in different topologies. The diverse paths generate different packets flows for the most part. 
<!--more-->

# Contivpp.io Packet Flows

The [PacketFlow Development Guide](https://github.com/contiv/vpp/blob/master/docs/dev-guide/PACKET_FLOW.md) serves up the details and descriptions of
paths traversed by request and response packets inside a Contivpp Kubernetes cluster under different situations.


