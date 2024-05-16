# Setup K8s using rke2
Similar to k3s, Rancher also provides a K8s distribution aiming datacenter use cases.

Their website provides more information with the differences with k3s:

https://docs.rke2.io/#how-is-this-different-from-rke-or-k3s

# Setup cluster using rke2

> Reference: https://docs.rke2.io/install/quickstart

1. Run the installer:

```bash
# Elevate to root
sudo su

# Run installer
curl -sfL https://get.rke2.io | sh -

# Enable service
systemctl enable rke2-server.service

# Start service
systemctl start rke2-server.service

# Exit running command with root
exit

# RKE2 ships with kubectl, but it does not add the folder to path
# Lets add it
PATH=$PATH:~/var/lib/rancher/rke2/bin/

# If we want to execute kubectl commands, we need to setup kubeconfig file
mkdir -p $HOME/.kube
sudo cp -i /etc/rancher/rke2/rke2.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Lets check our node
kubectl get nodes

# Output
# NAME     STATUS   ROLES                       AGE    VERSION
# nzl535   Ready    control-plane,etcd,master   8m3s   v1.28.9+rke2r1
```