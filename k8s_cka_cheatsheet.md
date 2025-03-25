# Imperative Commands:

**Starting with aliases**

```
alias kgpw="kubectl get pods -o wide"
alias kgns="kubectl get ns"
alias kgall="kubectl get all --all-namespaces"
```

**ETCD Commands:**

```
ETCDCTL_API=3 ./etcdctl version
```

**Commands:**

### Creating a pod with the labels assigned

```
kubectl run redis --image=redis:alpine --labels="tier=db"
```

### Creating a Pod and Exposing It as a Service in One Step

Documentation reference: https://notes.kodekloud.com/docs/CKA-Certification-Course-Certified-Kubernetes-Administrator/Core-Concepts/Solution-Imperative-Commands-optional.

```
kubectl run nginx --image=nginx --port=80 --expose=true

or

kubectl run httpd --image=httpd:alpine --port=80 --expose=true
```

### Creating Services when the existing resource is available(Pod, Deployment, ReplicaSet)

Reference documentation: https://notes.kodekloud.com/docs/CKA-Certification-Course-Certified-Kubernetes-Administrator/Core-Concepts/Solution-Imperative-Commands-optional#creating-services.

```
kubectl expose pod nginx --target-port=80 --port=80

or 

kubectl expose pod redis --port=6379 --name=redis-service
```

and, other examples:

```
kubectl expose rc nginx --port=80 --target-port=8000
kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000
kubectl expose pod valid-pod --port=444 --name=frontend
kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https
kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream
kubectl expose rs nginx --port=80 --target-port=8000
kubectl expose deployment nginx --port=80 --target-port=8000
```

### Creating Services from Scratch

```
kubectl create service clusterip my-cs --tcp=5678:8080
```

### Create Deployment

Deployment is same as ReplicaSet


```
kubectl create deployment webapp --image=kodekloud/webapp-color
kubectl scale deployment webapp --replicas=3

or

kubectl create deployment nginx --image=nginx:latest --replicas=3

or 

kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns

```

### Creating Namespace:

```
kubectl create namespace dev-ns
```


### Scenario-based question on Schedulers:

What if, if there is no `kube-scheduler`, how do you assign the pods to the specific nodes?

A: We need to assign this manually either using the `binding object` or, we can manually update the `pod-definition.yaml` with the `nodeName`.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
    name: nginx-pod
spec:
    nodeName: node01    ---- adding this nodeName manually
    containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - protocol: TCP
              port: 443
```


### Using Selectors to get the specific pods using the Labels:

```
k get all --selector="env=prod"

k get pod --selector="env=prod,bu=finance,tier=frontend"
```

### Taints and Tolerations:

#### Basics:

**Taints is for Nodes**

So,

```
# To add the taints to the specific node

kubectl taint node node01 spray=mortein:NoSchedule

# To remove the taint from the specific node

kubectl taint node node01 spray=mortein:NoSchedule-
kubectl taint node node01 spray-
```

**Tolerations is for Pods**


```yaml
# pod-definition-file.yaml
# Adding Tolerations for the above taints
apiVersion: v1
kind: Pod
metadata:
    name: nginx
spec:
    tolerations:
    -   effect: "NoSchedule"
        key: "spray"
        operator: "Equal"
        value: "mortein"
    containers:
    -   name: nginx
        image: nginx:alpine
```

### Labelling the nodes:

```
kubectl label nodes node01 size=large
```

#### Adding Node Selectors

Where the `pod-definition.yaml` looks like the below using the `nodeSelector`:

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  nodeSelector:
    size: large  # Direct key-value pair, no "matchLabels"
  containers:
    - name: nginx
      image: nginx:alpine

```

#### Node Affinity


```yaml
# Setting Node Affinity: Set Node Affinity to the deployment to place the pods on node01 only.

$ cat deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: blue
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: "color"
                operator: "In"
                values: 
                - "blue"
      containers:
      - image: nginx
        name: nginx
```

Create a new deployment named red with the nginx image and 2 replicas, and ensure it gets placed on the controlplane node only.

Use the label key - `node-role.kubernetes.io/control-plane` - which is already set on the controlplane node.

```yaml
$ cat controlplane.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: "node-role.kubernetes.io/control-plane"
                operator: "Exists"
      containers:
      - image: nginx
        name: nginx
```

### Adding DaemonSet 

```yaml
$ cat daemonset_eg.yaml 
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    tier: node
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: app
  template:
    metadata:
      labels:
        k8s-app: app
    spec:
      containers:
      -  image: registry.k8s.io/fluentd-elasticsearch:1.20
         name: fluentd-elasticsearch
```

### Static Pod Path

```
# default
/etc/kubernetes/manifests

or else, we can check the kubelet config file for `StaticPodPath`

something like this: /var/lib/kubelet/config.yaml
```

## Relationship and Important notes:

- Deployment and ReplicaSets are same not much difference, just the `kind`
- configmap and secrets are same, difference is just the `kind`
- If you have passed some variables or changes in the `/etc/kubernetes/manifests/*` to anyfile there, it bypasses the `scheduler` and updates the pods accordingly.
- Make use of `crictl` to understand what's happening on the pods if you don't have a `kubectl` as it relies on the `kube-apiserver`.




