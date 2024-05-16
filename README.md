# Getting started with Kubernetes
Getting started with Kubernetes presentation material

# Why Kubernetes?

Kubernetes is a logic next step on your journey to run containers in a production environment. Docker offer you an great developer experience, but does not scale well in a complex production deployment.

Kubernetes offers you a great abstraction of “where” the cluster is running: it can work on a in-house cluster, or maybe on a cloud provider (AKS)

Understanding how to setup a Kubernetes cluster help you understand what happens “under the hood” when using Kubernetes.

# Table of contents
- [Install Linux using WSL2](1-install-linux-wsl2-setup.md)
- [(Bonus) Install Linux using HyperV](2-install-linux-hyperv-setup.md)
- [Setup Kubernetes using `kubeadm`](3-setup-k8s-using-kubeadm.md)
- [Getting started with `kubectl`](4-kubectl-getting-started.md)
- [Creating and applying a manifest](5-kubectl-create-apply-manifest.md)
- [Exposing a deployment using a service](6-kubectl-expose-service.md)
- [(Bonus) Setup Kubernetes using k3s](7-setup-k8s-using-k3s.md)
- [(Bonus) Setup Kubernetes using RKE2](8-setup-k8s-using-rke2.md)