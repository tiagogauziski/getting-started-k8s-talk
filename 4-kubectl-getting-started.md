# Getting started with `kubectl`

## Running some `kubectl` commands

Lets get a feeling of our cluster

```bash
# Get everything running on the cluster
# -A or --all-namespaces will list everything useful seeing
kubectl get all -A

# Check all nodes associated to your cluster
kubectl get nodes

# Extract extra information from the cluster by running kubectl describe
kubectl describe node nzl535
```

Ok, we ran a bunch of commands, but nothing useful

Lets deploy something on it

```bash
# We are going to deploy a simple Pod running nginx
kubectl run nginx --image=nginx

# Lets check it, make sure it's running
kubectl get pods

# Output
# NAME    READY   STATUS    RESTARTS   AGE
# nginx   0/1     Pending   0          118s
```

Oh my, every time we deploy something, looks like something goes wrong. This is actually good. Give us an opportunity to troubleshoot our cluster.

To be honest, you should get used to troubleshooting 😀

Why is it on `pending` state?

Lets describe the `Pod` state to get more insights what is going on.

```bash
# Describe the Pod state
kubectl describe pod nginx

# Output
# ...
# Events:
#   Type     Reason            Age                  From               Message
#   ----     ------            ----                 ----               -------
#   Warning  FailedScheduling  11m                  default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
#   Warning  FailedScheduling  55s (x2 over 5m55s)  default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
```

The events part is giving us some insights: looks like there is no node available to this workload to be land.

But we just created this cluster!

Well, unfortunately, `kubeadm` setup the first node with a `taint` that prevents anything that is not important enough to run on our control plane node.

For our testing, the node taint is actually making our life worst: lets remove it!

```bash
# Lets run again a describe on the node
kubectl describe node nzl535

# Output
# ...
# Taints:             node-role.kubernetes.io/control-plane:NoSchedule
# ...

# Ok, we found the culplit, lets nuke it
kubectl taint nodes nzl535 node-role.kubernetes.io/control-plane:NoSchedule-

# Output
# node/nzl535 untainted

# Lets check out Pod again
kubectl get pods

# Output
# NAME    READY   STATUS    RESTARTS   AGE
# nginx   1/1     Running   0          30m
```

Our `Pod` is not running! Awesome!

Ok, out motto here is “trust but verify”. How can we check if the thing is actually running?

Lets check how we can make a HTTP call to our nginx instance:

```bash
# Lets get the IP address assigned to the Pod
# Note: "-o wide" gives us more information on the output table
kubectl get pods -o wide

# Output
# NAME    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
# nginx   1/1     Running   0          34m   10.244.0.4   nzl535   <none>           <none>

# Lets make call that IP using CURL on port 80
curl 10.244.0.4

# Output
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

Great! 

Lets enjoy our winning here.

Before we call it a day, lets delete the nginx `Pod`

```bash
kubectl delete pod nginx

# Output
# pod "nginx" deleted
```