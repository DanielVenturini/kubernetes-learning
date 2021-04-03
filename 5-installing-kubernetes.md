# Installing Kubernetes

There are some different cluster configurations. Once Kubernetes ecosystem is heavy, to use a business configuration is not the good way when running in a laptop to study it. So, we can consider a lightweight option when installing the Kubernetes. The major installations types are described below:

- **All-in-One Single-Node installation**: in this setup, the master and worker components are executed in the same machine. This setup is highly recommended when learning, development and testing and it is not recommended to be used in production. `Minikube` is the most famous program that implements this setup.
- **Single-Master and Multi-Worker installation**: there is a single-master node running a stacked etcd instance and there may be multiples workers nodes managed by the master.
- **Single-Master with Single-Node etcd and Multi-Worker installation**: in this setup, the etcd node is not the same node of the master (external etcd instance). There may be multiples workers managed by the master node.
- **Multi-Master and Multi-Worker installation**: there are multiples master node configured in High-Availability (HA) and each master node execute a stacked etcd instance. The etcd instance are also configured in an HA etcd cluster and there may be multiple workers.
- **Multi-Master with Multi-Node etcd and Multi-Worker installation**: there are multiple master node configured in HA with one external etcd node for each master node. There may be multiples workers nodes. This is the most advanced cluster configuration.

---

The most common installation tools available for all-in-one is the following:

- **[K3S](https://k3s.io/)**: lightweight Kubernetes cluster for local, cloud, edge, IoT deployments -- from Rancher.
- **[minikube](https://minikube.sigs.k8s.io/docs/)**: the most recommended tool for learners that are looking for a single-node installation.
- [**microk8s**](https://microk8s.io/): low-ops, minimal production Kubernetes, for devs, cloud, cluster, workstation, Edge and IoT -- from Canonical.
- [**kind**](https://kind.sigs.k8s.io/docs/): a multi-node Kubernetes cluster that requires Docker as a container runtime.
- [**Docker Desktop**](https://www.docker.com/products/docker-desktop): a Desktop application to pull Docker images that also includes a local Kubernetes cluster.

A [comparative](https://brennerm.github.io/posts/minikube-vs-kind-vs-k3s.html) between minikube, kind and k3s.

Kubernetes can be installed **on-prrmise VMs**, such as Vagrant, VMware vSphere, KVM, or **on-premise Bare Metal**, which is installed in a lot of different operating system. Also, there are some cloud options to install and execute a Kubernetes instance, the ones called Hosted Solutions. In this solution, the Kubernetes and others tools are managed by the provider, which generally charges by the cloud.

- [Alibaba Cloud Container Service for Kubernetes](https://www.alibabacloud.com/product/kubernetes) (ACK).
- [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (EKS).
- [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS).
- [DigitalOcean Kubernetes](https://www.digitalocean.com/products/kubernetes/).
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE).
- [IBM Cloud Services](https://www.ibm.com/cloud/kubernetes-service).
- [Oracle Cloud Container Engine for Kubernetes](https://www.oracle.com/cloud-native/container-engine-kubernetes/) (OKE).



There are also some installation tools available. This tools [are used to install and prepare](https://kubernetes.io/docs/setup/production-environment/tools/) the Kubernetes ecosystem:

- **[kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)**: it's the first-class citizen on the Kubernetes ecosystem and this is the most recommended method to bootstrap a single/multi-node production ready HA Kubernetes cluster.
- **[kubespray](https://github.com/kubernetes-sigs/kubespray)**: it can be used to install HA production ready Kubernetes cluster on AWS, GCE, Azure, OpenStack an so on. It's a Kubernetes Incubator project and it's available on most Linux distributions.
- **[kops](https://github.com/kubernetes/kops)**: it allows us to create, upgrade and maintain production-grade, HA Kubernetes cluster from the command line. AWS is officially supported, but GCE and OpenStack is in beta and VMware vSphere is alpha.
- [**The hard way**](https://github.com/kelseyhightower/kubernetes-the-hard-way): this is a hard way tutorial for learners that explain step-by-step how to bootstrap Kubernetes without script. This is useful to understand the low-level steps around the Kubernetes installation.

---

**Windows installation** is very limited to a workers nodes. Since Kubernetes v1.14, Windows was successfully introduced as a [supported production](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/) operation system, but only for worker nodes. There isn't way/plan to support a Windows as a control plane/master node. Only Windows Server 2019 is supported as a worker node.

---

You should now be able to:

- Discuss Kubernetes configuration options.
- Discuss infrastructure considerations before installing Kubernetes.
- Discuss infrastructure choices for a Kubernetes cluster deployment.
- review Kubernetes installation tools and resources.