# Services

The communication is crucial to microservices while running in the cluster. The [Service](https://kubernetes.io/docs/concepts/services-networking/service/) object abstracts the network access inside and outside the cluster. Services provides a single DNS entry and load balancing to a set of pods logically grouped and managed by a controller such as a Deployment, ReplicaSet, or DeamonSet. We will also learn about **kube-proxy** daemon, which provides access to services and is executed in each cluster's machine. Also, we will take a look in **service discovery** and **service types**.

---

### Services

Once the Pods are ephemeral in nature, when accessing the cluster through a Pods' IP address might be dangerous: if a Pod terminates, the IP address for a new Pod certainly changes. So, Pod's IP addresses must not be static.

<img src="https://courses.edx.org/assets/courseware/v1/3f0cf38178b8639549e3b78b7c634ba3/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/service-2.png" alt="pod-change-ip" style="zoom: 50%;" />

Services is the Kubernetes object that aims to provide a higher-level abstraction **grouping Pods and defining policies** to access them. This grouping is achieved via [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

We can set Labels and Selectors as a pair or **key/value** and we can set these labels in Pods. For those Labels, we can create a Service that specifies a set of those Labels and all Pods with these Labels will be managed by this Service.

Using Labels and Selectors, we can create a **logical group of Pods** that will be managed by a Service. There are two Services in the following example: where the first one matches all Pods with Label *app==frontend* and the second one matches Pods with Label *app==db*.

<img src="https://courses.edx.org/assets/courseware/v1/43deaf159772d06b10039d683640c244/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/Services2.png" alt="services-labels" style="zoom: 33%;" />

In the following example, the Service `nginx-service` matches the Pod `nginx-unique-pod` once **all the selectors in the Service matches the labels in the Pod**. If one of the selectors did not match any of the labels in the Pod, the Service will not match this Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unique-pod
  namespace: default
  labels:
    doesntmatter: once	# this label will not influence in the Service matches
    app: nginx
    env: production
spec:
  containers:
    - name: nginx-unique-pod
      image: nginx:latest
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    env: production
    app: nginx
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 80
      nodePort: 32134
  type: NodePort
```

By default, a Service receives a routable IP address that connects the Pod with the outside world. This IP address is called **ClusterIP**. User connects to the Service through the ClusterIP and the Service forwards the connection to Pods attached to it. Service also **provides a load balance**. When a Service attaches more than a single Pod, each Pod receives a unique IP address to be accessed (***endpoint***). The ClusterIP address and the endpoint address is automatically and specifically created and managed by the Service.

When the Service forwards the traffic, it holds all the incoming traffic from the port **port** and forwards to the port **targetPort** in which the Pod receives the traffic. In the previous example, an user can connect to a Pod in port **8000** and the Service forwards this traffic to port **80**, in which the Pod is executing. It is crucial the **targetPort** in Service matches the **port** in containerPort.

### kube-proxy

[kube-proxy](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies) is a deamon that runs on each cluster node and watches the API server for addition, updates, and removal of Services and endpoints. kube-proxy applies the Service configuration to enable traffic routing to an exposed application running in Pods. When a new Service is configured, kube-proxy configures iptables rules on each node to capture the traffic from its ClusterIP. Then, the traffic can be forwarded to any endpoint in the cluster.

![kube-proxy](https://courses.edx.org/assets/courseware/v1/f6184f33a4c81a2c59eb9c28bf79c3ae/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/kubeproxy.png)



---

Once Services are essential for communication among cluster's node, it is helpful to be able to discover the Services addresses at runtime. There are two ways to do it:

- **Environment variables**: when a Pod starts, Kubernetes loads several information about all active Services in the cluster.  For example, for a Service called `redis-master` that exposes the port **6379** and its **ClusterIP** is **172.17.0.6**, the environment loaded in the Pod is:

  - REDIS_MASTER_SERVICE_HOST=172.17.0.6
  - REDIS_MASTER_SERVICE_PORT=6379
  - REDIS_MASTER_PORT=tcp://172.17.0.6:6379
  - ...

  This solution is not the best, once new Services is not loaded for older Pods.

- [**DNS**](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/): Kubernetes has a DNS add-on that provides DNS records for all the cluster. Using DNS, all Services may be easily found over the cluster. The FQDN is the following: *`service`.`namespace`.**svc**.**cluster**.**local***. Services/Pods within the same namespace can find other Services just using the `service`. Services/Pods from the other namespace can find other Services using the *`service.namespace`*. Using DNS records is the most common and highly recommendation solution.

---

The Service can have different scopes that fits the project desire. The Service can :

- only be accessible within the cluster.
- be accessible from within the cluster and the external world.
- maps to an entity which resides either inside or outside the cluster.

We can set the Service type in the **ServiceType** property when creating the Service.

Services can expose single Pods, ReplicaSets, Deployments, DaemonSets, and StatefulSets.