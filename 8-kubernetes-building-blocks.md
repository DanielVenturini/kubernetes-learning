# Kubernetes Building Blocks

In this chapter, we will explore the [Kubernetes object model](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) and describe some of its fundamental building blocks, such as **Pods**, **ReplicaSets**, **Deployments**, **Namespaces**, etc. We will also discuss the essential role of **Labels** and **Selectors** in a microservices driven architecture as they logically group decoupled objects together.

---

There are an [API Convention](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md), which describes technically each of the Kubernetes terms.

The Kubernetes object model is very rich and can represent several **persistent entities** in the Kubernetes cluster. Those entities describe the containerized applications we are running, the nodes that are deploying, the application resources and a lot of another configurations. Kubernetes uses these entities to represent the state of your cluster. In the **spec** section we can describe the ***desired state*** of the object and Kubernetes manages the **status** section for objects, where the **current state** of the object is. Then, the Control Plane continually tries to match the desired status (`spec`) to current state (`status`). For example, a Deployment was configured to execute three replicas (desired state -- `spec`), but in a moment one replica falls down, the current state (`status`) is different from the desired state and the control plane tries to fix it.

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

**Important:** Pods should never be created directly using a manifest file of kind Pod. Pods are not self-healing and, hence, a Pod cannot recreate an instance of itself it there may be an error. The reliable way to create Pods is using a Deployment which configures a [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) controller to manage the [Pod's lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/).

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

### ReplicationController

Although no longer a recommended controller, a [ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) ensures a specified number of Pods running at same time. A ReplicationController constantly verifies the desired state of managed application and ensures the current state will match. If the number of desired Pods is smaller than the actual number, the ReplicationController instances new Pods; If greater, the ReplicationController terminates them.

The default way to manage Pods is using a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), which configures a [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) controller to manage [Pod's lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/).

### ReplicaSets

A [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) is the next-generation of the legacy ReplicationController as well there are some advantages in the ReplicaSet use against the ReplicationController such as the ReplicaSet supports both equality/set-based selectors whereas ReplicationController supports only equality-based selectors.

A ReplicaSet aims to keep a set of replica Pods running in a given time as the desired state. For a specific number of replicas described in the `spec.replicas`, the ReplicaSet creates as much replicas as specified. Both Pods are identically and they run the same container image once they are cloned from the Pod template -- `spec.template`. The information about the ReplicaSet that created a Pod is in `metadata.ownerReferences`.

<img src="https://courses.edx.org/assets/courseware/v1/bdb9c27c39d027fc22133456a2882fa6/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/Replica_Set1.png" alt="replica-set-working" style="zoom:33%;" />

There is a very interesting thing about the ReplicaSet and labels.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: to-fill-unique-pod
  namespace: default
  labels:
    delete: once
    app: to-fill
    env: production
    # arch: x86 # with label commented, ReplicaSet will not match this pod
spec:
  containers:
    - name: to-fill-unique-pod
      image: danielventurini/to-fill:1.0
      ports:
        - containerPort: 8080
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: to-fill-metadata
  namespace: default
  labels:
    delete: once
    app: to-fill
spec:
  replicas: 2
  selector:
    matchLabels:
      delete: once
      app: to-fill
      env: production
      arch: x86
  template:
    metadata:
      labels:
        delete: once
        app: to-fill
        env: production
        arch: x86
    spec:
      containers:
        - name: to-fill-unique-pod
          image: danielventurini/to-fill:1.0
          ports:
            - containerPort: 8080
```

In the first part of the manifest above, we create a Pod. This Pod has specific labels in `metadata` field. Then, in the second part of the manifest above, we create a ReplicaSet that instances two Pod's replicas. All the Pods that this ReplicaSet manages is labeled as `spec.selector.matchLabels[*]`. The Pod in the first part of the manifest has three of four labels managed by the ReplicaSet. When we apply this file, Kubernetes creates a Pod and a ReplicaSet with two Pod's instance, hence, we have three Pods. If we add the label `arch: x86` to the Pod in the first part of the manifest, the ReplicaSet would acquire this Pod and manage it. So, ReplicaSet  would notice that there are three Pod's instance in the current state whereas the desire state (`spec.replicas`) specifies only two instances. Finally, the ReplicaSet would delete the first Pod and keep only two replicas as well. It happens because the Pod's template in ReplicaSet (`spec.template`) will be the same as `metadata` in Pod, so, both Pods are the same Kubernetes object.

We can also delete a ReplicaSet and leave all of the Pods unchanged, that is, orphan.

The Deployment is the high-level concept that manages a ReplicaSet. A Deployment provides a set of complementary features to orchestrate Pods. When a Deployment creates Pods, it also creates a ReplicaSet to manage these Pods. **It is not recommended to create and manage a ReplicaSet or a Pod directly**, unless there may be some custom orchestration resources/updates or do not require updates at all.

### Deployments

A [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is a Kubernetes object that aims to create and manage Pods and ReplicaSets. The DeploymentController is part of the master's node controller managers and it ensures the current state matches the desired state. A Deployment allows for seamless applications **updates and rollbacks** through **rollouts** and **rollbacks**.

In the following image, a Deployment creates three ReplicaSets, which ones create two Pods. Each ReplicaSet reads the Deployment's template in field `spec.template` and they manage the Pods according the template.

<img src="https://courses.edx.org/assets/courseware/v1/5b2c8cbd6bed63c68f6a7d2566615d7f/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/Deployment_Updated.png" style="zoom: 25%;" />

The Deployment allows the application to easily receive push updates. When a update is pushed to container, the Deployment **triggers a new ReplicaSet** for the new container and it creates a new **Revision**, as in the following image, and this transition from the previous ReplicaSet to the new ReplicaSet is a Deployment **rolling update**. The new ReplicaSet is scaled to the required number of Pods wheres the previous ReplicaSet is scaled down to 0 Pods and **this ReplicaSet is "stored" for further rollbacks**. All updates in specifics properties of the Pod's template in Deployment metadata triggers a rolling update and, hence, a new revision.

<img src="https://courses.edx.org/assets/courseware/v1/979a990d505485a3ad502836ca5f1078/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/ReplikaSet_B.png" alt="deployment-rolling-update" style="zoom:25%;" />

In the above image, the previous state used the container image **nginx:1.7.9**, but when there was an update in the Pod's template in Deployment metadata to image **nginx:1.9.1**, the Deployment saved the **Revision 1** and created the **Revision 2**.  If there is something wrong with the update, the Deployment can be safety rolled back to Revision 1. In this case, the Revision 1 becomes the Revision 3, and there is no more Revision available, just Revision 2 and 3. Kubernetes stores up to 10 Revision set.

When there is a deploy, a new ReplicaSet is scaled up and the previous ReplicaSet is scaled down. This previous ReplicaSet's Pod Template is saved with the desired state as 0 replicas. Then, it creates a new Revision, that we can see using through the `rollout` command. Also, the previous ReplicaSet's Pod Templates can be seen yet (#1,#2). We then can see the amount of Revisions that can be rolled back (#3). We also can see detailed the ReplicaSet's Pod Template for a specific revision number (#4). Finally, we can restore a previous ReplicaSet's Pod Template (#5), then, **the Revision 3 become the latest Revision**.

```bash
kubectl -n mynamespace get replicasets	# 1
kubectl -n mynamespace describe replicasets replset-name	# 2
kubectl -n mynamespace rollout history deployment deplname	# 3
kubectl -n mynamespace rollout history deployment deplname --revision=3	# 4
kubectl -n mynamespace rollout undo deployment deplname --to-revision=3	# 5
```

Revisions are not a Deployment exclusive, but we can rollback DaemonSets and StatefulSets.

### Namespaces

A [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) is a logical cluster separator that allow multiple users and teams to work on the same Kubernetes cluster. The names of the resources/objects created in a Namespace **is unique in that Namespace**, but across the Namespace in the cluster. There are some pre-built namespaces, such as `kube-system`, `kube-public` and the basic one, the `default` namespace. The namespace `kube-public` is really public, it means this namespace provides insecurity access and anyone can read it. Other important namespace is the `kube-node-lease`, which holds **node lease objects** used for node heartbeat data.

---

You should now be able to:

- Describe the Kubernetes object model.
- Discuss Kubernetes building blocks, e.g. Pods, ReplicaSets, Deployments, Namespaces.
- Discuss Labels and Selectors.