# Setup K8s using k3s
Now that we know how to setup a Kubernetes cluster using `kubeadm` tool, lets check other options out there

Rancher has some K8s compliant distributions that works pretty well and are easy to use.

> Reference: https://docs.k3s.io/

# Installing using k3s

First, check out their documentation: https://docs.k3s.io/quick-start

To setup a Kubernetes cluster using k3s is pretty easy:

```bash
curl -sfL https://get.k3s.io | sh -
```

Once the installation is done, you can access the cluster via:

```bash
sudo k3s kubectl get node
```

If you want to use `kubectl` directly, you might need some extra steps:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Then we can run the same commands as before:

```bash
kubectl get nodes

# Output
# NAME     STATUS   ROLES                  AGE     VERSION
# nzl535   Ready    control-plane,master   9m47s   v1.29.4+k3s1
```