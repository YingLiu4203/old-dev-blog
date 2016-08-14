---
layout: post
title: Basic Kubernetes
categories:
- Development
tags:
- Cloud-Computing
---

## K8s Architecture
First we need a set of computers, physic or virtual, to urn K8s, these computers are called [**node**][node] in K8s. Nodes are often organized as a cluster to provide high availability and scalability. In each node there is a [*kubelet*][kubelet] node agent, a [*kube-proxy*][kube-proxy]network proxy and a **docker daemon** to run one or more [*pods*][pod]. In K8s, a pod consists of a set of docker containers. A pod is the minimum management unit to create, run and schedule.

To manage a K8s cluster, we need a control plane and a distributed storage solution. The control plane has a number of management components. Currently they all run in a single *master* node but in the future may run in a cluster. A distributed storage provides data sharing for the whole cluster. Management components include scheduler, controller-managers (replication, namespace, service account etc). An [*etcd*][etcd] service provide shared persistent configuration and service discovery.

## Pod, Deployment and Service

K8s has many types of resources. Pod, deployment and service are most commonly used resources. Resources can be identified by their names or their labels. [Labels][label] are just key/value pairs that are attached to resources. Labels support two types of selectors: equality-based and set-based.

Equality-based selectors:

```code
environment = production
tier != frontend
```
Set-based selectors:

```code
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

`kubectl` uses `-l` option to specify label selectors. For example:

```sh
$ kubectl get pods -l environment=production, tier=frontend
$ kubectl get pods -l 'environment in (production, qa)'
```

### Pod
According to [Pod Document][pod]:

>pods are the smallest deployable units of computing that can be created and managed in Kubernetes.

>A pod (as in a pod of whales or pea pod) is a group of one or more containers (such as Docker containers), the shared storage for those containers, and options about how to run the containers. Pods are always co-located and co-scheduled, and run in a shared context. A pod models an application-specific “logical host” - it contains one or more application containers which are relatively tightly coupled — in a pre-container world, they would have executed on the same physical or virtual machine.

Commands to manage pods:

```sh
# create a pod from its defintion file, kubectl run can create a pod from a docker image.
$ kubectl create -f pod-def.yaml

# list all pods
$ kubectl get pod

# list a specific pod
$kubectl get pods pod-name

# describe a pod
$kubectl describe pods pod-name

# delete pods
$ kubectl delete pod pod-name

```

 ### Deployment

 According to [Deployment document][Deployment]:
 >A Deployment provides declarative updates for Pods and Replica Sets..... You can define Deployments to create new resources, or replace existing ones by new ones.

From the above description, we know that deployment is used to create pots and specify the number of replicates. It allows online update of running pods.

### Service
The [Service document][service] defines a **Kubernetes Services** as:

> A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service. The set of Pods targeted by a Service is (usually) determined by a Label Selector.

> For Kubernetes-native applications, Kubernetes offers a simple Endpoints API that is updated whenever the set of Pods in a Service changes. For non-native applications, Kubernetes offers a virtual-IP-based bridge to Services which redirects to the backend Pods.

In the above description, an **EndPoint** is simply a set of pod IP addresses and ports. It is used to map service IP:ports to backend pod IP:ports.

There are three types of services:

* ClusterIP: only reachable inside a cluster.
* NodePort: reachable outside a cluster by **<NodeIP>:NodePort**. The choice of backend pod is random or based on client-IP session affinity.
* LoadBalancer: ask cloud provider for a load balancer for **<NodeIP>:NodePort** for each node.

### Service Discovery


### K8s API
One can access K8s APIs using `kubectl` command or their REST API.  Details are in the [cluster api access introduction][cluster-api]. The REST API can be accessed using either a kubectl proxy using kubectl or direct http request.

To know the location and credentials of a cluster, use this command: `kubectl config view`.

In K8s, the nodes, pods and services all have their own IPs. There is no need to access node IP in an application. All pods should be accessed from a service ip. A service with type `NodePort` or `LoadBalancer` can be reachable outside the cluster. A `ClusterIP` service is the default setting and is only visible inside a cluster.



[node]: https://github.com/kubernetes/kubernetes/blob/master/docs/design/architecture.md
[kubelet]: http://kubernetes.io/docs/admin/kubelet/
[kube-proxy]: http://kubernetes.io/docs/admin/kube-proxy/
[pod]: http://kubernetes.io/docs/user-guide/pods/
[cluster-api]: http://kubernetes.io/docs/user-guide/accessing-the-cluster/
[etcd]: https://coreos.com/etcd/docs/latest/
[deployment]: http://kubernetes.io/docs/user-guide/deployments/
[label]: http://kubernetes.io/docs/user-guide/labels/
[service]: http://kubernetes.io/docs/user-guide/services/
