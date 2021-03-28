# Kubernetes Architecture

In this chapter we'll explore the **Kubernetes architecture**, the components of its **control plane**, the **master** and **worker nodes**, the cluster state management with **etcd** and the network setup requirements. We'll also learn about the **Container Network Interface (CNI)**, as Kubernetes' network specification.

Basically, the Kubernetes architecture is compose of one or more node master and one or more node workers. The most common configuration is one master and two workers and each node most likely will execute in a different host or different virtual machines.

![kubernetes-cluster](https://courses.edx.org/assets/courseware/v1/51120ad23b216a6946e3c4ebef2106bf/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/arch-1.19-components-of-kubernetes.svg)

The master node provides a running environment for the Control Plane, which manages the state of the Kubernetes cluster (master and workers) through the several agents. These agents work inside the Control Plane. The communication among the nodes in Kubernetes is done through a command line interface (**kubectl**), a web user-interface (**dashboard**) or through application programming interface (**api**). Once the master node becomes the most important node in the cluster, it must never stop suddenly. To ensure High-Availability (HA) in Kubernetes, the master node may be replicated in two or more nodes, each one executing independently, and they exchange information to ensure that in all time the Control Plane is working in one of those nodes. If one master node fails, other one get up and does not let the Kubernetes environment break down.

The Kubernetes cluster's data is stored in [**etcd**](https://etcd.io/), which is a distributed key-value database that stores all **data about the state of the cluster**. Data about configuration is stored in **etcd**, but the application data and client data aren't. [etcd can be configured to ensure HA](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/) in the following ways:

- **stacked etcd topology**:  each control plane runs an instance of `kube-apiserver`, `kube-controller-manager` and `kube-scheduler`. The `kube-apiserver` is exposed to worker nodes using the **load balancer**. Each control plane creates a local instance of etcd that communicates only with `kube-apiserver`. So, the etcd instance and the control plane are running in the same node.![stacked-etcd](https://cdn.ttgtmedia.com/rms/onlineImages/itops-stacked_etcd-f_mobile.jpg)
- **external etcd topology**: as in the stacked topology, each control plane contains the `kube-apiserver`, `kube-scheduler` and `kube-controller-manager`. The `kube-apiserver` is exposed to worker nodes using a load balancer and exchanges information with the etcd instance. However, the etcd instance is not running in the same machine of the control plane. There should be a dedicated node to execute the instances of etcd. ![external-etcd](https://cdn.ttgtmedia.com/rms/onlineImages/itops-external_etcd-f_mobile.jpg)

The stacked etcd topology is the default one and the simplest, but if a node fails, the control plane and the etcd instance fail too. The external etcd topology is the most reliable, once the control plane and the etcd instance do not depend on the same node, but this topology requires twice nodes more than stacked one.

Each control plane has an `API server`, a `scheduler`,  `controller managers` and a `data store`. In addition, the master node runs a `container runtime`, a `node agent` and a `proxy`. Each of them are described below:

- `API Server`: it's the main point of communication in the Kubernetes. The `kube-apiserver` receives RESTful requests from users, operators and external agents and processes these requests. API Server reads data from the etcd and uses these data to processes the requests. After, the API Server updates the data in etcd and the API Server is the unique component that exchange information with etcd. The API Server is highly configurable and customizable and the Kubernetes architecture can have a secondary API Server.
- `Scheduler`: 

