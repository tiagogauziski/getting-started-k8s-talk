# Setup K8s using kubeadm

## Setup the node

- Lets setup it to install Kubernetes

```powershell
# Lets check if swap is enabled
free -h

# Output
# tiago@NZL535:~$ free -h
#                total        used        free      shared  buff/cache   available
# Mem:            15Gi       618Mi        14Gi       2.7Mi       341Mi        14Gi
# Swap:             0B          0B          0B

# Because swap is set to 0B, means that it's disabled
# In case it's not 0B, you can disable it using 
sudo swapoff -a

# Source: https://kubernetes.io/docs/setup/production-environment/container-runtimes/
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Source: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
# Add some packages to help us setup the cluster
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Download the signing key for the apt repository
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes apt repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update apt package index, install tools and hold upgrades
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Lets install container runtime - we are going with containerd
# Source: https://github.com/containerd/containerd/blob/main/docs/getting-started.md#customizing-containerd
sudo apt-get update && sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

## Install cluster using `kubeadm`

We will continue on the terminal you have been using

```powershell
# It's time to use kubeadm
sudo kubeadm init --pod-network-cidr 10.244.0.0/16

# Once it's done, you should see an output similar to this:
# Your Kubernetes control-plane has initialized successfully!
# 
# To start using your cluster, you need to run the following as a regular user:
# 
#   mkdir -p $HOME/.kube
#   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
#   sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 
# Alternatively, if you are the root user, you can run:
# 
#   export KUBECONFIG=/etc/kubernetes/admin.conf
# 
# You should now deploy a pod network to the cluster.
# Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
#   https://kubernetes.io/docs/concepts/cluster-administration/addons/
# ...
# kubeadm join 172.31.216.121:6443 --token 33twou.ez4wvad4rbo7n6ni \
#        --discovery-token-ca-cert-hash sha256:dedb42ed80961e0bdd6070a52fddab7c853a047ea5cc2183ebd4804089db7d55
```

## If you have a worker node

```powershell
# Notice that when we setup the cluster, on the message at the end, it has a `kubeadm join` command
# We are going to use it now
kubeadm join 172.31.216.121:6443 --token 33twou.ez4wvad4rbo7n6ni \
        --discovery-token-ca-cert-hash sha256:dedb42ed80961e0bdd6070a52fddab7c853a047ea5cc2183ebd4804089db7d55
```

## Check our cluster setup

```powershell
# Let's check our cluster setup
kubectl get nodes

# Output
# E0514 20:33:58.475404    2936 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
```

Why can’t se connect to our cluster. 

This error is actually easy to fix. We just need to move kube config created by `kubeadm` to our user home folder (~/.kube/config)

In fact, all the instructions are added on the end of the `kubeadm` setup:

```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Once we run it, lets rerun `kubectl get nodes` again:

```bash
kubectl get nodes

# Output
# NAME     STATUS     ROLES           AGE    VERSION
# nzl535   NotReady   control-plane   6m6s   v1.30.0
```

Awesome. We got some better results. But not enough. It still reports the node as `NotReady`. Why?

## Installing a CNI

Networking is a big part of the Kubernetes experience. Kubernetes `kubeadm` tool will provide you a cluster, but leaves up to you to select your favorite CNI plugin (there are many options out there)

For us to get something going, we will use flannel (one of the easiest option out there)

```bash
# Reference: https://github.com/flannel-io/flannel#deploying-flannel-with-kubectl
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Easy!

Let’s check out cluster health again:

```bash
kubectl get nodes

# Output
# NAME     STATUS   ROLES           AGE   VERSION
# nzl535   Ready    control-plane   15m   v1.30.0
```

Awesome, our cluster is healthy now!