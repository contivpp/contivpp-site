+++
title = "K8s Services"
type = "components"
summary = "How Contivpp.io handles Kubernetes Services"
weight = "2"
+++

Service is a Kubernetes abstraction providing a convenient single entry point
of access to a group of pods. In other words, a service can be thought of 
as a dynamic loadbalancer for a set of pods (and the containers living inside
them), automatically managed by the K8s framework itself. 
The set of Pods targeted by a Service is (usually) determined by a Label Selector. Contivpp.io supports 
the mapping of k8s services to the VPP dataplane.


[Details](https://github.com/contiv/vpp/blob/master/docs/dev-guide/SERVICES.md)



