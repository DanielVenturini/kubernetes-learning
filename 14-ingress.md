# Ingress

Using the NodePort *ServiceType* may be trick sometimes, and the LoadBalancer requires a heavy infrastructure to be supported. Ingress API is another way to expose our application that represents another layer of abstraction, deployed in front of the Service API resources.

---

### Ingress

We can have a lot of routing rules within a Service. We can have several Services, therefore, so much routing rule in our cluster. When we can decouple the routing rules, we can use an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/). Ingress is an API object that manages external access in the cluster and may provide load balancing, SSL and name-based virtual hosting.

Ingress operates in the front of a Services. When an HTTP or an HTTPS request reaches the Ingress, it can route the request to a specific Service based on the routing rules. For example, every `/last-position` request might be routed to a specific Service and, thus, to a specific application.

The following is an example of a **Name-Based Virtual Hosting** Ingress object. All requests for both `blue.example.com` and `gree.example.com` would go to the same Ingress endpoint. All incoming requests on the host `blue.example.com` would be redirected to the service `webserver-blue-svc` endpoint on port 80 and all incoming requests on the host `green.example.com` would be redirected to service `webserver-green-svc` endpoint on port 80, too.

```yaml
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  name: virtual-host-ingress
  namespace: default
spec:
  rules:
  - host: blue.example.com
    http:
      paths:
        - backend:
          service:
            name: webserver-blue-svc
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
  - host: green.example.com
    http:
      paths:
        - backend:
          service:
            name: webserver-green-svc
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
```



![ingress](https://courses.edx.org/assets/courseware/v1/15999c8685f44d6fe3a9f8f9e15b1d66/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/ingress-nbvh.png)

---

There may be the **Fanout** Ingress object. In this kind of Ingress, the determination factor is the path on requests. All incoming requests to `example.com` will be redirected according to path. In the following example, requests on path `/blue` (`example.com/blue`) would be redirected to service `webserver-blue-svc` on port 80 whereas requests on path `/green` (`example.com/green`) would be redirected to service `webserver-blue-svc` on port 80, too.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fan-out-ingress
  namespace: default
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /blue
        backend:
          service:
            name: webserver-blue-svc
            port:
              number: 80
        pathType: ImplementationSpecific
      - path: /green
        backend:
          service:
            name: webserver-green-svc
            port:
              number: 80
        pathType: ImplementationSpecific
```

![fanout-ingress](https://courses.edx.org/assets/courseware/v1/f92ffefe5a16b804d584804b44017796/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/ingress-fanout.png)

### Ingress Controller

The Ingress resource does not forward any requests by itself, it merely accepts the definitions of traffic routing rules. The Kubernetes object that aims with traffic redirection is the [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/), which is a reverse proxy that really does the traffic routing based on rules defined in the Ingress resource. Without an Ingress Controller, the Ingress does not do anything on the cluster.

> Ingress Controllers are also known as Controllers, Ingress Proxy, Service Proxy, Reverse Proxy, etc.

There are a bunch of [Ingress Controllers available for Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). The most 