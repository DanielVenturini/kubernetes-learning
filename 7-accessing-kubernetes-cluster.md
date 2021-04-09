# Accessing K3S/Minikube

In this chapter, we will learn about different methods of accessing a Kubernetes cluster.



We can use a variety of external clients or custom scripts to access our cluster for administration purposes. We will explore **kubectl** as a CLI tool to access the Kubernetes cluster, the **Kubernetes Dashboard** as a web-based user interface to interact with the cluster and `curl` command with proper credentials to access the cluster via APIs. All Kubernetes cluster can be access via a CLI, a Dashboard or an API Server.

### Kubectl

The most known and important CLI tool is [kubectl](https://kubectl.docs.kubernetes.io/), the one used to manage cluster resources and applications. After required credentials, kubectl can be used remotely from anywhere. kubectl connects to the API Server in control plane on the master node and, after authentication, kubectl can manage all the cluster.

The [kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) file is a file that stores all the necessary information about cluster, users, namespaces and informations required to kubectl to authenticate and access the API Server.  By default, kubectl looks for this file in `~/.kube` directory. The kubeconfig file contains three main sections, *cluster*, *context* and *name* and the file can be access through the command `kubectl config view`.

### Dashboard

The [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) is the Web-based User Interface (Web UI) used to access the Kubernetes master and manage all the cluster. Dashboard can be started by `kubectl proxy` command, which one authenticates with the API Server and makes Dashboard available. When we run `kubectl proxy`, the kubectl creates a gateway between the API Server and localhost:8001, so, we can access the Dashboard on localhost.

- Dashboard URL: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
- Get token: `kubectl -n kubernetes-dashboard get secret kubernetes-dashboard -o yaml`, however, this token must be decoded in base64.
- Get token without base64: `kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")`


### API Server

 Also, the Kubernetes cluster can be managed through the **APIs from CLI**. The API Server is the main component of Kubernetes control plane. API Server exposes all the APIs, which ones allow users and operators to interact with the cluster. There are a lot of different APIs in Kubernetes.![apis](https://courses.edx.org/assets/courseware/v1/7ebcb514203a3af89dc1599625779c1f/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/api-server-space_.jpg)

The HTTP API directory tree of Kubernetes can be divided into tree main group types. We can access these API endpoint through either by CLI or Dashboard:

- **Core group (/api/v1)**: in this subtree, there are all of the most important and basic resources in cluster, such as Pods, Nodes, Namespaces, ConfigMaps, Secrets and so on.
- **Named Group (/apis/$NAME/$VERSION)**:  the objects in this group are referenced via their different level of stability and support, where *Alpha level* may be dropped in any time, *Beta level* is well tested, but the objects' semantics may change in subsequent release and *Stable level*, which the object is well consolidated.
- **System-wide**: this group contains all the system-wide API endpoints, such as */healthz*, */metrics*, */apis* and so on.

After executing the command `kubectl proxy`, the kubectl creates a gateway between the API Server and the localhost:8001. It allows us to securely access all the API. Then, we can get access the APIs through the localhost:

- curl http://localhost:8001/api/
- curl http://localhost:8001/api/v1
- curl http://localhost:8001/apis/authorization.k8s.io/v1beta1
- curl http://localhost:8001/healthz
- curl http://localhost:8001/metrics

---

You should now be able to:

- Compare methods to access a Kubernetes cluster.
- Configure kubectl for Linux.
- Access the Kubernetes cluster from the Dashboard.
- Access Kubernetes via APIs.