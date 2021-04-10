# Kubernetes Building Blocks

In this chapter, we will explore the [Kubernetes object model](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) and describe some of its fundamental building blocks, such as **Pods**, **ReplicaSets**, **Deployments**, **Namespaces**, etc. We will also discuss the essential role of **Labels** and **Selectors** in a microservices driven architecture as they logically group decoupled objects together.

---

There are an [API Convention](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md), which describes technically each of the Kubernetes terms.

The Kubernetes object model is very rich and can represent several **persistent entities** in the Kubernetes cluster. Those entities describe the containerized applications we are running, the nodes that are deploying, the application resources and a lot of another configurations. Kubernetes uses these entities to represent the state of your cluster. In the **spec** section we can describe the desired state of the object and Kubernetes manages the **status** section for objects, where the actual state of the object is. Then, the Control Plane continually tries to match the desired status (spec) to current state (status). For example, a Deployment was configured to execute three replicas (desired state -- spec), but in a moment one replica falls down, the current state (status) is different from the desired state and the control plane tries to fix it.

The next is an example of an Deployment object's configuration manifest:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.11
        ports:
        - containerPort: 80
```

- `apiVersion`: the first required field and specifies which API endpoint in the API Server we want to connect to. 
- `kind`: the second required field and specifies the object type.
- `metadata`: the third required field, metadata holds the object's basic information, such as name, namespaces, labels, etc.
- `spec`: the fourth required field describes the desired object state. All pods from this object model manifest are created using the Pod Template defined in **spec.template**. When a Pod inherits from this object model manifest, the Pod retains its **metadata** and **spec**, but the **apiVersion** and **kind** are replaced by **template**.
- `spec.template.spec`: this field defines the desired state **of the Pod**.

The previous manifest doesn't have any **state** field because this field is created by the Kubernetes system.

### Pod

A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) is the smallest, simplest and the widely known Kubernetes object. It represents an instance of the application **when it is deployed**, so, a Deploy object can create a Pod. A Pod contains one or more containers, which ones share the network namespaces, the IP address, that is, all the containers inside a Pod share the same IP address, and all the containers have access to mount the same external storage. Usually, a Pod contains only a single container, which one is the application.![pods-containers](https://courses.edx.org/assets/courseware/v1/ccc5ba54a8a06ac2a87fe447bb53dcf1/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/pods-1-2-3-4.svg)

Pods are extremely ephemeral, that is, they may have short life-cycle and they may be easily created, deleted and recreated, but they do not have the capability to self-heal themselves. This is the reason they are always related with a controller, such as a Deployments, ReplicaSets, etc, and these controller handle Pod's replication, fault tolerance, self-healing, etc. The Pod's metadata is inherited from the controller that creates the Pod, using the Pod Template in `spec.template`.

The code below is an example of a Pod's manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.11
    ports:
    - containerPort: 80
```

### Labels

[Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) are simply a **key-value pairs** attached to Kubernetes object to create a concise management. They are just used as a easily way to organize and select objects. Controllers use Labels to logically group together decoupled objects, rather than using object's name or ID.

<img src="https://courses.edx.org/assets/courseware/v1/6669997d43534cbd2f251a57ebe0587c/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/Labels.png" alt="labels" style="zoom: 33%;" />

Labels do not provide uniqueness by Pods, but we can set a bunch with the same label, instead. Labels is the Kubernetes way to group Pods. We can insert one or several labels in the `spec.template.labels.[*]` and then, we can search Pods by those labels (#1). We can even set the same label on Pods in different namespaces, and get them with `--all-namespaces` option (#2):

```bash
kubectl get pods -l env=qa	#1
kubectl get pods --all-namespaces -l heritage=Helm	#2
kubectl get pods --all-namespaces -l heritage!=Helm	# get all that aren't Heml
```

However, we must pay attention that the **labels in `metadata.labels.[*]` will not be inherited by Pods**, just the ones in `spec.template.labels.[*]`.

There is also the [Label Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) way to select Pods based on their labels. It allows us to search in a great way than the early way. There is the **equality-based selectors**, that works as before using the operators **=**, **==**, and **!=**. There is also the **set-based selectors**, that provides a way to search using a bunch of tags using **in**, **notin**, **exist/does not exist** operators.