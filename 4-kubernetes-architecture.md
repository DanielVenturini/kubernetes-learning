# Kubernetes Architecture

In this chapter we'll explore the **Kubernetes architecture**, the components of its **control plane**, the **master** and **worker nodes**, the cluster state management with **etcd** and the network setup requirements. We'll also learn about the **Container Network Interface (CNI)**, as Kubernetes' network specification.

Basically, the Kubernetes architecture is compose of one or more node master and one or more node workers. The most common configuration is one master and two workers and each node most likely will execute in a different host or different virtual machines.

![kubernetes-cluster](https://courses.edx.org/assets/courseware/v1/51120ad23b216a6946e3c4ebef2106bf/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/arch-1.19-components-of-kubernetes.svg)

The master node provides a running environment for the Control Plane, which manages the state of the Kubernetes cluster (master and workers) through the several agents. These agents work inside the Control Plane. The master node encapsulates the client applications inside Pods -- the smallest unit of Kubernetes, that contain one or more containers -- and execute it into one of the workers.

The communication among the nodes in Kubernetes is done through a command line interface (**kubectl**), a web user-interface (**dashboard**) or through application programming interface (**API**). Once the master node becomes the most important node in the cluster, it must never stop suddenly. To ensure High-Availability (HA) in Kubernetes, the master node may be replicated in two or more nodes, each one executing independently, and they exchange information to ensure that in all time the Control Plane is working in one of those nodes. If one master node fails, other one get up and does not let the Kubernetes environment break down.

## [etcd](https://etcd.io/)

The Kubernetes cluster's data is stored in etcd, which is a distributed key-value database that stores all **data about the state of the cluster**. Data about configuration is stored in **etcd**, but the application data and client data aren't and **the data is never replaced, just appended**. etcd can be managed through its CLI, the **etcdtl**. etcd is based in [Raft Consensus Algorithm](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14) that creates a bunch of machines and one of them is the master; if one machine falls down, the workload is balanced among the remain nodes. [etcd can be configured to ensure HA](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/) in the following ways:

- **stacked etcd topology**:  each control plane runs an instance of `kube-apiserver`, `kube-controller-manager` and `kube-scheduler`. The `kube-apiserver` is exposed to worker nodes using the **load balancer**. Each control plane creates a local instance of etcd that communicates only with `kube-apiserver`. So, the etcd instance and the control plane are running in the same node and the etcd shares information among the others etcd's instances.![stacked-etcd](https://cdn.ttgtmedia.com/rms/onlineImages/itops-stacked_etcd-f_mobile.jpg)
- **external etcd topology**: as in the stacked topology, each control plane contains the `kube-apiserver`, `kube-scheduler` and `kube-controller-manager`. The `kube-apiserver` is exposed to worker nodes using a load balancer and exchanges information with the etcd instance. However, the etcd instance is not running in the same machine of the control plane. There should be a dedicated node to execute the instances of etcd.![external-etcd](https://cdn.ttgtmedia.com/rms/onlineImages/itops-external_etcd-f_mobile.jpg)

The stacked etcd topology is the default one and the simplest, but if a node fails, the control plane and the etcd instance fail too. The external etcd topology is the most reliable, once the control plane and the etcd instance do not depend on the same node, but this topology requires twice nodes more than stacked one.

## Master Node (control plane) components

Each master node has a control plane, which has an `API server`, a `scheduler`,  `controller managers` and a `data store`. In addition, the master node runs a `container runtime`, a `node agent` and a `proxy`. Each of them are described below:

- **API Server**: it's the main point of communication in the Kubernetes. The `kube-apiserver` receives RESTful requests from users, operators and external agents and processes these requests. API Server reads data from the etcd and uses these data to processes the requests. After, the API Server updates the data in etcd and the API Server is the unique component that exchange information with etcd. The API Server is highly configurable and customizable and the Kubernetes architecture can have a secondary API Server.
- **Scheduler**:  it's the component in the control panel that aims in which worker the workload object, such as Pods, will be execute. To do it, the Scheduler, through API Server, gets data from the etcd about the workers status. The scheduler verifies each worker status' data and chose the worker that fits the requirements for the workload object. For example, a pod may require to be executed in a worker with ssd, so, the scheduler gets all status' data from the workers and verifies the label **disk==ssd** and selects a worker that fits all required resources. The outcome decision is communicated back to API Server that delegates the workload deployment with other control plane agents. The scheduler is also highly configurable.
- **Controller managers**: they ensure the desire state of the workers will be constantly in execution. Controller manages run as a watch-loops verifying whether the state of the workload objects, such as a pod, is the exactly as their definition files. The desired state's data come from the configuration files and the execution state's data come from the etcd through the API server. When one or more workers become unavailable, the `kube-controller-manager` ensures the pod counts will continue as expected. The `cloud-controller-manager` runs controllers responsible to interact with underlying infrastructure when a node becomes unavailable. The `cloud-controller-manager` manages storage volumes, load balancing and routing.

## Worker Node Components

The worker nodes execute in a different machine from the master node. The worker node has the following components:

- **Container Runtime**: Kubernetes does not provide a native container runtime, but Kubernetes supports a lot of different ones. These containers runtime should be installed in the worker nodes, where the Pods will be executed. There are some containers runtime:
  - [docker](https://www.docker.com/): uses **containerd** as a container runtime, and it's the most popular.
  - [CRI-O](https://cri-o.io/): a lightweight container runtime for Kubernetes that supports Docker image registries.
  - [containerd](https://containerd.io/): a simple and portable container runtime providing robustness.
  - [frakti](https://github.com/kubernetes/frakti#frakti): a hypervisor-based container runtime for Kubernetes.
- **Node Agent (`kubelet`)**: the kubelet is an agent that runs on each worker and communicates with the API Server in control plane on master node. The kubelet receives the PODs definitions from the API Server and communicates with the container runtime to manages the container. The kubelet connects with the container runtime through the [Container Runtime Interface (CRI)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md). The **shim** application provides an abstraction layer between the kubelet and the container runtime and, once the shim is where the runtime is connected, Kubernetes doesn't need to be recompiled when a the container runtime is changed. The CRI implements two services: **ImageService**, which ensures about all the image-related operations; and **RuntimeServices**, which ensures about all the Pod and container-related operations. Finally, the kubelet monitors the health and resources of Pods running containers. ![kubelet-cri](https://courses.edx.org/assets/courseware/v1/ab209f7c32ceb17ed43dcf6b66056cea/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/CRI.png)
- **Proxy (`kube-proxy`)**: 
- **Addons**:
  - DNS: 
  - Dashboard user interface: 
  - cluster-level monitoring: 
  - logging: 