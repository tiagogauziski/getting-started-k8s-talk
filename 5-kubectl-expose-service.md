# Exposing a deployment using services
We will explore how we can we expose our deployment via Service

# Kubernetes `Service`

Let’s say we create a webserver and deploy it to our cluster using `Deployment` manifest.

How can we expose it to the world?

We can do it via a Kubernetes `Service` . Services acts very similar to a tradition load balancer. It will spread your load to all the pods behind your deployment.

Check the documentation for more information: https://kubernetes.io/docs/concepts/services-networking/service/

Let’s create a `Service` to complement our deployment

```bash
# Lets write a service manifest 
cat <<EOF | tee test-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  type: NodePort
  selector:
    app: hello
  ports:
  - name: http-port
    protocol: TCP
    port: 80
EOF

# Apply the manifest
kubectl -n my-testing apply -f test-service.yaml

# Lets check out service
kubectl -n my-testing get service

# Output
# NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# test-service   NodePort   10.106.205.87   <none>        80:31801/TCP   2m13s

# We can now call the IP address of our service
# tiago@NZL535:~$ curl 10.106.205.87
# <html><head><title>HTTP Hello World</title></head><body><h1>Hello from hello-deployment-57cc58b976-f74ck</h1></body></html
# tiago@NZL535:~$ curl 10.106.205.87
# <html><head><title>HTTP Hello World</title></head><body><h1>Hello from hello-deployment-57cc58b976-wsfrq</h1></body></html
```

Look, that is quite interesting, our service got a special IP address that we can use (10.106.205.87)

Kubernetes also has a DNS server running (thanks to those CoreDNS pods running)

Our Pods internally will be able to access DNS entries instead of IP addresses. 

Lets run a trick to run a container and attach to a shell from inside the container:

```bash
# Run an alpine Pod and connect to it via shell
kubectl run -i --tty alpine --image=alpine --restart=Never --rm -- sh

# Lets use wget (similar to cURL) to call our DNS entry
wget -qO- test-service.my-testing.svc.cluster.local

# Output
<html><head><title>HTTP Hello World</title></head><body><h1>Hello from hello-deployment-57cc58b976-867ds</h1></body></html

# Lets check where this DNS entry is pointing to
nslookup test-service.my-testing.svc.cluster.local

# Output
# Server:         10.96.0.10
# Address:        10.96.0.10:53
# 
# 
# Name:   test-service.my-testing.svc.cluster.local
# Address: 10.106.205.87
```

Even better, because we created a `NodePort` type of our `Service` , we can access it even outside the cluster, using the the node IP address and the additional port that our service provided us

```bash
# Lets get the node internal IP Address
kubectl get nodes -o wide

# Output
# NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION                       CONTAINER-RUNTIME
# nzl535   Ready    control-plane   26h   v1.30.0   172.31.216.121   <none>        Ubuntu 24.04 LTS   5.15.146.1-microsoft-standard-WSL2   containerd://1.7.12

# Let's get the port of out NodePort Service
kubectl -n my-testing get service

# Output
# NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# test-service   NodePort   10.106.205.87   <none>        80:31801/TCP   21m
```

Ok, we found that:

- Our node IP is: 172.31.216.121
- Our service node port is: 31801

```bash
# Open a new Terminal and call the IP Address on that port
curl 172.31.216.121:31801

# Output
<html><head><title>HTTP Hello World</title></head><body><h1>Hello from hello-deployment-57cc58b976-f74ck</h1></body></html
```

You can also open your browser and type that same URL, it will work just the same!

# Tidy up

Before we finish, let’s delete our namespace, as we don’t need the resources any longer

```bash
kubectl delete namespace my-testing

# Output
# namespace "my-testing" deleted
```