+++
title = "K8s Policies"
type = "components"
summary = "How Contivpp.io handles Kubernetes Policies."
weight = "1"
+++

Kubernetes network policies specify how groups of pods are
allowed to communicate with each other and other network endpoints. Each
policy is represented as an instance of the K8s resource `NetworkPolicy`. 
A policy uses labels to select a grouping of pods and specifies a list of rules
that determine which traffic is allowed to and from the selected pods.
Contivpp.io implements the Kubernetes Network API, including 
egress policies and IP blocks.

[Details](https://github.com/contiv/vpp/blob/master/docs/dev-guide/POLICIES.md)

