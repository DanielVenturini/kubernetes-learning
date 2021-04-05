# Minikube - A local Kubernetes cluster

Minikube is the most popular way to install all-in-one Kubernetes cluster in a virtual machine (VM) locally on our workstation. The latests version of minikube has been developed to support multi-node Kubernetes cluster, but this feature is still experimental.

To execute, minikube requires a [Type-2 hypervisor](https://en.wikipedia.org/wiki/Hypervisor#Classification) or a Container Runtime. Minikube uses [libmachine](https://github.com/docker/machine/tree/master/libmachine) to invoke the Hypervisor that creates a single-node VM, or the Container Runtime to run the Container that hosts the cluster. Then, [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) is used to provision the Kubernetes cluster. Finally, VirtualBox is the Minikube's original hypervisor driver.

Minikube requires some resources that must be available in the machine:

- **VT-x/AMD-v virtualization** must be enable.
- **kubectl**: this is the binary used to access and manage any Kubernetes cluster. It is installed through Minikube or can be installed separately.
- **Type-2 Hypervisor or Container Runtime**: one of them is required to create a VM. The most common applications are [VirtualBox](https://www.virtualbox.org/wiki/Downloads), [KVM2](https://www.linux-kvm.org/page/Main_Page), [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) or Docker runtime.
  - If use `--driver=none` on **Linux distribution**, Minikube will not create/use a VM, but Minikube will run all the Kubernetes components directly on the host OS. If use `--driver=podman` or `--driver=docker`, Minikube will run all the Kubernetes components in the top of the container runtime.
- **Internet connection**: at least the first Minikube run requires an internet connection to download packages, dependencies, pull images and others actions.

VirtualBox will be our default hypervisor.

There are several ways to [install Minikube](https://minikube.sigs.k8s.io/docs/start/) and to [install VirtualBox hypervisor](https://www.virtualbox.org/wiki/Linux_Downloads) on Linux. We can use [CRI-O](https://cri-o.io/) as a Container Runtime, once Docker [isn't compatible with OCI](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/). and will be removed in the following Kubernetes releases -- actually, the dockershim will be removed, therefore, the Docker support will be removed.

---

You should now be able to:

- Understand Minikube.
- Install Minikube on local Linux, macOS and Windows -- I have my doubt in those two last systems.
- Verify the local installation.