# Creating and applying a manifest
We so far only played with kubectl imperative commands (like `kubectl run` for example).

Lets explore how we can create Kubernetes manifests and apply it on the cluster.

## How to create a namespace

Kubernetes has the concept of a namespace, which is a logical separator of resources within a cluster. 

When you are retrieving a namespaced resource (like `Pod` or `Deployment`), when not specified, it will use `default` :

- For example: when running `kubectl get pods` command, it’s the same as running `kubectl get pods -n default`

Lets get going then…

```bash
# Create a namespace for our experiment
kubectl create namespace my-testing

# Output
# namespace/my-testing created
```

## Deploy deployment manifest

The nice thing about a Kubernetes manifest, is that it allow you to write a file that specify the state of your cluster. You will provide all details about the deployment, and the manifest file itself is a document that can be version controlled.

How can we get started with a manifest? The best website to get knowledge about Kubernetes: Kubernetes documentation website (https://kubernetes.io/docs/home/)

In this article(https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment), it gives you simple manifest to get started. Lets copy and paste on file (with little tweaks) 

```bash
# Write a file from our output
cat <<EOF | tee test-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
  labels:
    app: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello-web
        image: strm/helloworld-http
        ports:
        - containerPort: 80
EOF

# Lets apply the manifest on our namespace
kubectl -n my-testing apply -f test-deployment.yaml

# Output
# deployment.apps/test-deployment created

# Lets check all resources created by our deployment
kubectl get all -n my-testing

# Output
# NAME                                    READY   STATUS    RESTARTS   AGE
# pod/hello-deployment-57cc58b976-867ds   1/1     Running   0          28s
# pod/hello-deployment-57cc58b976-f74ck   1/1     Running   0          28s
# pod/hello-deployment-57cc58b976-wsfrq   1/1     Running   0          28s
# 
# NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/hello-deployment   3/3     3            3           28s
# 
# NAME                                          DESIRED   CURRENT   READY   AGE
# replicaset.apps/hello-deployment-57cc58b976   3         3         3       28s
```

Ok, bunch of things running there.

Let’s try get more information about our running pods.

```bash
# Get more details from our pods
kubectl -n my-testing get pods -o wide

# Output
# NAME                                READY   STATUS    RESTARTS   AGE    IP            NODE     NOMINATED NODE   READINESS GATES
# hello-deployment-57cc58b976-867ds   1/1     Running   0          107s   10.244.0.21   nzl535   <none>           <none>
# hello-deployment-57cc58b976-f74ck   1/1     Running   0          107s   10.244.0.19   nzl535   <none>           <none>
# hello-deployment-57cc58b976-wsfrq   1/1     Running   0          107s   10.244.0.20   nzl535   <none>           <none>

# Lets get one of those IP address and cURL it
curl 10.244.0.21

# Output
# <html><head><title>HTTP Hello World</title></head><body><h1>Hello from hello-deployment-57cc58b976-867ds</h1></body></html
```

We will continue our journey on the next article.