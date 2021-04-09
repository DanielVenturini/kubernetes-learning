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
- `spec`: the fourth required field describes the desired object state. All pods from this object model manifest are created using the Pod Template defined in **spec.template**. When a Pod inherits from this object model manifest, the Pod retains its **metadata** and **spec**, but the **apiVersion** and **king** are replaced by **template**.
- `spec.template.spec`: this field defines the desired state of the Pod.

The previous manifest doesn't have any **state** field because it is created by the Kubernetes system.