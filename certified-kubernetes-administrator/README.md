11th Dec - 
# Certified Kubernetes Adminitrator

https://www.cncf.io/certification/cka

Mode of delivery: Online
Online Proctor will be available

----
### Prerequesites

It tests your hands-on skills.

3 hours will be given. You'll be able to refer to the official K8s documentation. 

In late-2023:

With the latest version of the exam, it is now only 2 hours. The contents of this course have been updated with the changes required for the latest version of the exam.

Below are some references:

Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/

Exam Curriculum (Topics): https://github.com/cncf/curriculum

Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

Exam Tips: http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD

Head over to this link to enroll in the Certification Exam. Remember to keep the code – 20KODE – handy to get a 20% discount while registering for the CKA exam with Linux Foundation.

----
### Core Concepts
----

#### Cluster Architecture

ETCD - Is a database that stores data in a key-value format.

Kube-scheduler - It identifies the right node to add more containers or manage containers if the nodes resources are high.

Controller-Manager:
- Node-controllers - Takes care of nodes, responsible for adding more nodes and manage nodes.
- replica-controller - It is responsible to handle the desired number of containers are running all-times in a replicaset or replication group.

Kube-apiserver - It is the management of the K8s cluster. It responsible for orchestrating all operations between the k8s cluster and the external users. And, it exposes APIs to communicate within the components within the Master node or the worker nodes itself.

Container Runtime Engine or Container Runtime Interface
- Docker
- ContainerD
- Rocket - rkt
- Crio

These are very important to run the containers on the worker nodes and master nodes.

Kubelet - It is an agent that runs on each node in a K8s cluster. It listens for instructions from the Kube-APIServer and Kube-APIServer talks with the Kubelet to get the status of the nodes and containers. It is the captain manages containers on the nodes.

Kube-Proxy - It is a service that allows connections and network between the applications that are deployed in the worker nodes.

----

### ETCD for Beginners

##### What is ETCD?

ETCD is a reliable key-value store that is secure.

##### What is a Key-Value Store?

##### How to get statred quickly?

Download the binary, extract and run the etcd service that starts a sercice that listens on 2379(default). The default client is `etcdctl` so you can make use of it store and manage the etcd store. 

For example:

```
./etcdctl set key1 value1 -- works v2.0
./etcdctl get key1 -- works v2.0
```

> ETCD Versions

v0.1. Aug 2013
v0.5 Dec 2014
v2.0 Feb 2015
v3.1 Jan 2017
     Nov 2018 0 CNCF incubation

##### How to find out ETCDCTL is configured to work with what versions?

We would need to pass the `--version` as an [option] in version 2.
```
$ ./etcdctl --version
etcdctl version: 3.3.11
API version: 2
```

> [!NOTE] With the release of version 3.4, the default API version is set to 3.

##### How to set/change ETCDCTL API version for ETCDCTL v3

We can mention the `API version` via the `environmental variable`  - `ETCDCTL_API` 
```
$ ETCDCTL_API=3 ./etcdctl version
etcdctl version: 3.3.11
API version: 3.3
```

Also, instead of mentioning it each time, we can export it to the enviroment variable for the `profile`

```
$ export ETCDCTL_API=3

then,

$ ./etcdctl version
etcdctl version: 3.3.11
API version: 3.3
```

----

### ETCD for Kubernetes

What is the role of the ETCD in the K8s?

ETCD is reponsible to journal each and every tasks and operations that are happening in the K8s cluster i.e. it records the details of the pods, nodes, configs, secrets, accounts, roles, bindings and more. Every information that is executed via the `kubectl` command is being fetched from the `ETCD` server.

##### Additional information about ETCDCTL Utility

`ETCDCTL` is the `CLI tool` used to interact with ETCD.

`ETCDCTL` can interact with `ETCD Server` using 2 API versions – `Version 2` and `Version 3`. By default it’s set to use `Version 2`. **Each version has different sets of commands.**

For example, ETCDCTL `version 2` supports the following commands:

```
$ etcdctl backup
$ etcdctl cluster-health
$ etcdctl mk
$ etcdctl mkdir
$ etcdctl set
```

Whereas the commands are different in `version 3`

```
$ etcdctl snapshot save
$ etcdctl endpoint health
$ etcdctl get
$ etcdctl put
```

To set the right version of API set the environment variable `ETCDCTL_API` command

```
$ export ETCDCTL_API=3
```

When the API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.

Apart from that, you must also specify the path to certificate files so that `ETCDCTL` can authenticate to the `ETCD API Server`. The certificate files are available in the `etcd-master` at the following path. We discuss more about certificates in the security section of this course. So don’t worry if this looks complex:

```
--cacert /etc/kubernetes/pki/etcd/ca.crt
--cert /etc/kubernetes/pki/etcd/server.crt
--key /etc/kubernetes/pki/etcd/server.key
```

So for the commands, you must specify the `ETCDCTL API` version and path to `certificate` files. Below is the final form:

```
$ kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"
```

> [!NOTE]
> Only when the ETCD key-value store is updated, any operations that are considered as completed or else, it is NOT.
> --advertise-client-urls https://${etcd_server}:2379 -- This is the default port that ETCD_server listens and this is important.

```
# Important command to display the keys stored in the etcd server - you would need to run this command inside the etcd-master pod

$ kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only

# K8s stores data in a specific directory structure, being the /root directory is `/registry`
# Example:

/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io

```
----

### Kube-API Server

General responsibilities of the `kube-api` server:

- Authenticate User
- Validate request
- Retrieve data from ETCD datastore
- Update ETCD
- Scheduler
- Kubelet

> [!NOTE]
> When using the `kubeadm` tool, it deploys kube-api server on our behalf and it does the heavy-lifting but, to understand how it all configured, you would need to download the binary of the `kube-apiserver` and add the configuration flags/parameters to connect to the cluster accordingly.

#### Commands that you can use to find the kube-apiserver

```
---- kubeadm setup ----
# When using `kubeadm` it deploys the kube-apiserver as a pod

$ kubectl get pods -n kube-system

To get the configurations of the kube-apiserver, we can check:

$ cat /etc/kubernetes/manifests/kube-apiserver.yaml

---- Non-kubeadm setup ----

# When deploying it from scratch using binary, where it is a non-kubeadm setup:

$ cat /etc/systemd/system/kube-apiserver.service

# Or, you can use the process command to check the parameters passed to run the kube-apiserver

$ ps -aux | grep -e "kube-apiserver"
```
----

### Kube Controller Manager

It manages various controllers in a k8s cluster and it has its own responsibilities:

- Node controllers
    * Monitoring the status of the nodes, and tries to add the nodes if they are failed.
    * Checks every `5s` with the nodes whether the node is healthy or NOT. If the node is unhealthy and does not respond to the healthcheck, then it waits for `40s` to consider the node is `unhealthy` and `unreachable`. After `5m`, the node-controller will move the pods in those that nodes to a new one i.e. create new pods in the node(s) that are healthy.

```
Node Monitor Period: 5s
Node Monitor Grace Period: 40s
Pod Eviction Timeout: 5m
```
- Replication controllers
   * Monitoring the status of the pods, and tries to maintain the desired number of pods are available within the replicaset at any time. If any pods fail, it will make sure to create a new one.

There are many-more controllers in the K8s cluster:

- Deployment-controller
- Namespace-controller
- Endpoint-controller
- Cronjob
- Job-controller
- PV-Protection-controller
- Service-Account-contoller
- Stateful-Set
- Replicaset
- Node-controller
- PV-Binder-controller
- Replication-controller and many more ...

All the above controller(s) are packaged into a single process called `Kube-controller-manager`.

#### Commands that you can use to find the kube-controller-manager

```
---- kubeadm setup ----
# When using `kubeadm` it deploys the kube-controller-manager as a pod

$ kubectl get pods -n kube-system

To get the configurations of the kube-controller-manager, we can check:

$ cat /etc/kubernetes/manifests/kube-controller-manager.yaml

---- Non-kubeadm setup ----

# When deploying it from scratch using binary, where it is a non-kubeadm setup:

$ cat /etc/systemd/system/kube-controller-manager.service

# Or, you can use the process command to check the parameters passed to run the kube-controller-manager

$ ps -aux | grep -e "kube-controller-manager"
```
----

### Kube Scheduler

Scheduler is only responsible for `DECIDING` which pods goes on which nodes and it does not deploy/create those pods in the nodes.

> Breaking down how it decides or takes decision that which nodes are the best to deploy/create a pod?

Let's take an example of a pod A, that got resource requirements of 10 CPUs and there are 4 nodes with 4 CPUs, 6 CPUs, 12 CPUs, and 16 CPUs. So, below are the methods that it follows:

- "Filter's the nodes" that got `greater than or equal` CPUs so, in this case, it will be just 2 nodes that got 12 CPUs and 16 CPUs available.
- "Ranks the nodes" that got free CPUs once the pod is deployed/created. In this scenario, there will be just 2 CPUs left in Node A and 6 CPUs left in Node B.
- Now, it decides that Node B is the best node that the pod can be deployed and informs the `Kubelet` to create/deploy the pods.

#### Commands that you can use to find the kube-scheduler

```
---- kubeadm setup ----
# When using `kubeadm` it deploys the kube-scheduler as a pod

$ kubectl get pods -n kube-system

To get the configurations of the kube-scheduler, we can check:

$ cat /etc/kubernetes/manifests/kube-scheduler.yaml

---- Non-kubeadm setup ----

# When deploying it from scratch using binary, where it is a non-kubeadm setup:

$ cat /etc/systemd/system/kube-scheduler.service

# Or, you can use the process command to check the parameters passed to run the kube-scheduler

$ ps -aux | grep -e "kube-scheduler"
```
----

### Kubelet

`Kubelet` is like a captain on the ship. There are sole point of contact of the master ship and send reports of the ship to the master. Basically, once the `kube-scheduler` decides the best node to deploy/create the pods or containers. It will pass its information to `kubelet` via `kube-apiserver`. This `kubelet` immediately gets the information from the `kube-apiserver` and informs the `container runtime` i.e. `Docker` or `containerD` to pull the images of the containers from the repository and deploy it.

> [!CAUTION]
> `kubeadm` tool does not deploy `kubelet` as a pod or create it by itself. We need to manually install & run the `kubelet` service on the worker nodes.

#### Commands that you can use to find the kubelet

```
---- kubeadm setup ----

NO Kubeadm setup

---- Supports only non-kubeadm setup ----

# When deploying it from scratch using binary, where it is a non-kubeadm setup:

$ cat /etc/systemd/system/kubelet.service

# Or, you can use the process command to check the parameters passed to run the kubelet

$ ps -aux | grep -e "kubelet"
```
----

### Kube Proxy

pod network - weave net

`kube-proxy` runs on each node as a process and it's job is to look for new services like `clusterIP`, `NodePort`, and `LoadBalancer` as these are virtual components and everytime a new service is created, it makes sure to create IP rules so that it can forward the traffic to the backend pods within the nodes. It uses `IP-Table` rules in each node of the cluster and forwards traffic.

#### Commands that you can use to find the kube Proxy

```
---- kubeadm setup ----
# When using `kubeadm` it deploys the kube-proxy as a pod, infact this is a `daemonset`

$ kubectl get pods -n kube-system

$ kubectl get daemonset -n kube-system

To get the configurations of the kube-scheduler, we can check:

$ cat /etc/kubernetes/manifests/kube-proxy.yaml

---- Non-kubeadm setup ----

# When deploying it from scratch using binary, where it is a non-kubeadm setup:

$ cat /etc/systemd/system/kube-proxy.service

# Or, you can use the process command to check the parameters passed to run the kube-proxy

$ ps -aux | grep -e "kube-proxy"
```
----

### Pods

- What are pods?

Pods are the smallest `object` that we can create within the K8s cluster. It's basically K8s cluster(consider, it's a single node cluster) > Node > Pod > Containers. So, you create containers within the pod and scale the pods while you want to scale the containers.

To create a pod from the command line, use the command:

Create an NGINX Pod

kubectl run nginx --image=nginx

As of version 1.18, kubectl run (without any arguments such as --generator ) will create a pod instead of a deployment.

To create a deployment using imperative command, use kubectl create:

kubectl create deployment nginx --image=nginx

Kubernetes Concepts – https://kubernetes.io/docs/concepts/

Pod Overview- https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/

#### YAML and how to construct it?

Starting with key-value pairs:

```
# All key-values pair(s) have space after the colon.
fruit: apple
liquid: water
gas: oxygen
number: one
```

Arrays

```
fruits:
-  apple
-  mango
-  orange
-  banana
```
So, all items in the array should follow the `-` hyphen in front and with equal number of spaces. Note, it is not restricted to have a only 2 or 3 spaces after hyphen but to keep it legible and need, 2 or 3 should work.


Dictionaries/Map

```
fruit:
   - - - - - - - - - - -
- | Banana:             |
  |   calories: 100g    |
  |   fat: 20g          |   <----- Dictionaries
  |   carbs: 20g        |
- | Apple:              |
  |   calories: 200g    |
  |   fat: 2g           |
  |   carbs: 100g       |
   - - - - - - - - - - -
```

So, dictionaries are the properties of the `object` where in the above example, the object is banana and its properties are calories, fat, carbs. So, for dictionaries, you would need to give the key and place the objects below with a space.

#### Pods with YAML

Please note, when creating the pods, the K8s cluster should have `4 ROOT LEVEL properties`:

```
#pod-definition.yml

apiVersion:
kind:
metadata:
spec:
```

The above ones are the mandatory properties.

Now, let's create an example `pod-definition.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: app-dev
  labels: 
    type: front-end
    app: development
spec:
  containers:
    - name: nginx-container
      image: nginx
```

Use the below command to create the pod in the K8s cluster:

```
kubectl create -f pod-definition.yml
```

Use the below command to get the pods:

```
kubectl get pods
```

And,

```
kubectl describe pod <pod_name>
```


Also, you can make use of the commands to quickly perform a dry-run.

```
kubectl run nginx --image=nginx --dry-run=client -o yaml 

# Response/Output

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

We can save its output to a YAML file and can rerun it quickly like
kubectl run nginx --image=nginx --dry-run=client -o yaml >> output.yaml
```
---

### Replication Controller

Replication controllers allows us to run multiple instances of single pod in the cluster.

There are two terms:
[1] Replication Controller - Older terminology
[2] Replica Set - Latest

Major difference between [1] and [2] is the:

```
selector:
  matchLabels:
    type: front-end
```

The `selectors` are NOT mandatory for the replication controllers but, it is mandatory for the replicasets.

#### Scaling of Replicas

Commands used here:
```
$ kubectl create -f replicaset-definition.yaml

$ kubectl get replicaset

$ kubectl delete <replicaset_name>     ----> This deletes all the pods underlying the replicasets

$ kubectl replace -f replicaset-definition.yml     ---> This command will delete the pod and recreate it instead of we deleting once and creating it again.

$ kubectl scale --replicas=6 -f replicaset.yml
```

Below is the replicaset and replication controller yaml definition file:

```
# Replication Controller

apiVersion: v1
kind: ReplicationController
metadata:
  name: app-rc
  labels: 
    type: replication_controller
spec:
  template:
    metadata:
      name: app-dev
      labels: 
        type: front-end
        app: development
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3

# Response/Output:

% kubectl get pods                                        
NAME                   READY   STATUS              RESTARTS   AGE
app-rc-9gbgl           0/1     ContainerCreating   0          3s
app-rc-dlb6z           0/1     ContainerCreating   0          3s
app-rc-mcv78           0/1     ContainerCreating   0          3s
app-replicaset-cjfmx   1/1     Running             0          113s
app-replicaset-fmzn6   1/1     Running             0          113s
app-replicaset-j99z5   1/1     Running             0          113s

% kubectl get pods
NAME                   READY   STATUS              RESTARTS   AGE
app-rc-9gbgl           0/1     ContainerCreating   0          6s
app-rc-dlb6z           1/1     Running             0          6s
app-rc-mcv78           1/1     Running             0          6s
app-replicaset-cjfmx   1/1     Running             0          116s
app-replicaset-fmzn6   1/1     Running             0          116s
app-replicaset-j99z5   1/1     Running             0          116s

% kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
app-rc-9gbgl           1/1     Running   0          9s
app-rc-dlb6z           1/1     Running   0          9s
app-rc-mcv78           1/1     Running   0          9s
app-replicaset-cjfmx   1/1     Running   0          119s
app-replicaset-fmzn6   1/1     Running   0          119s
app-replicaset-j99z5   1/1     Running   0          119s
```


```
# Replicaset definition file

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: app-replicaset
  labels: 
    type: replication_set
spec:
  template:
    metadata:
      name: app-dev
      labels: 
        type: front-end
        app: development
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end

# Response/Output:

% kubectl get pods
NAME                   READY   STATUS              RESTARTS   AGE
app-replicaset-cjfmx   1/1     Running             0          6s
app-replicaset-fmzn6   0/1     ContainerCreating   0          6s
app-replicaset-j99z5   0/1     ContainerCreating   0          6s

% kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
app-replicaset-cjfmx   1/1     Running   0          10s
app-replicaset-fmzn6   1/1     Running   0          10s
app-replicaset-j99z5   1/1     Running   0          10s
```

#### After Scaling:

```bash
% kubectl scale --replicas=6 -f replicaset-definition.yaml 
replicaset.apps/app-replicaset scaled

% kubectl get pods                                        
NAME                   READY   STATUS              RESTARTS   AGE
app-rc-9gbgl           1/1     Running             0          86s
app-rc-dlb6z           1/1     Running             0          86s
app-rc-mcv78           1/1     Running             0          86s
app-replicaset-82b88   0/1     ContainerCreating   0          3s
app-replicaset-b8djw   0/1     ContainerCreating   0          3s
app-replicaset-cjfmx   1/1     Running             0          3m16s
app-replicaset-d6tcb   1/1     Running             0          3s
app-replicaset-fmzn6   1/1     Running             0          3m16s
app-replicaset-j99z5   1/1     Running             0          3m16s

% kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
app-rc-9gbgl           1/1     Running   0          95s
app-rc-dlb6z           1/1     Running   0          95s
app-rc-mcv78           1/1     Running   0          95s
app-replicaset-82b88   1/1     Running   0          12s
app-replicaset-b8djw   1/1     Running   0          12s
app-replicaset-cjfmx   1/1     Running   0          3m25s
app-replicaset-d6tcb   1/1     Running   0          12s
app-replicaset-fmzn6   1/1     Running   0          3m25s
app-replicaset-j99z5   1/1     Running   0          3m25s
```

#### Delete replicaset

```bash
% kubectl get replicaset 
NAME             DESIRED   CURRENT   READY   AGE
app-replicaset   6         6         6       5m8s

% kubectl delete replicaset app-replicaset
replicaset.apps "app-replicaset" deleted

% kubectl get pods                        
NAME           READY   STATUS    RESTARTS   AGE
app-rc-9gbgl   1/1     Running   0          3m57s
app-rc-dlb6z   1/1     Running   0          3m57s
app-rc-mcv78   1/1     Running   0          3m57s

% kubectl get replicaset                  
No resources found in default namespace.
```

As we mentioned above, deleting the `replicaset` will also delete the pods as you in the above output.

#### Delete Replication Controller

```bash
% kubectl get replicationcontroller
NAME     DESIRED   CURRENT   READY   AGE
app-rc   3         3         3       9m27s

% kubectl delete replicacontroller app-rc
error: the server doesn't have a resource type "replicacontroller"

% kubectl delete replicationcontroller app-rc
replicationcontroller "app-rc" deleted

% kubectl get pods
No resources found in default namespace.
```

Now, we created pods via the `replicaset` and `replicationcontroller` and deleted them.

#### Additional Commands:

```bash

$ kubectl get pods - Lists all pods in the namespace.  
$ kubectl get replicaset - Lists all ReplicaSets in the namespace.  
$ kubectl describe replicaset <replica-set-name> - Shows details of a specific ReplicaSet.  
$ kubectl describe pod <pod-name> - Displays detailed information about a pod.  
$ kubectl delete pod <pod-name> - Deletes a specific pod.  
$ kubectl create -f <replicaset-definition-file>.yaml - Creates a ReplicaSet from a YAML file.  
$ kubectl delete replicaset <replica-set-name> - Deletes a specific ReplicaSet.  
$ kubectl edit replicaset <replica-set-name> - Modifies a ReplicaSet in real-time.  
$ kubectl delete pods <pod-name-pattern> - Deletes pods matching a pattern.  
$ kubectl scale replicaset <replica-set-name> --replicas=<number> - Scales the ReplicaSet to a specified number of replicas.  

# Then, to delete all the pod in one go:

$ kubectl delete pods --all
$ kubectl delete pods -l <label-key>=<label-value>
$ kubectl delete pods --all -n <namespace>

# Important, when in doubt, you can use:


$ kubectl explain replicaset
$ kubectl explain replicationcontroller
$ kubectl explain pod

So, the above explain command will give us the details of what to use.

# shorthand:

$ kubectl get rs - For replicasets
$ kubectl get deploy - For deployments
```

---

### Deployments

```bash
$ kubectl create -f deployment-definition.yaml
deployment.apps/app-deployment created
```
```bash
$ kubectl get all 

NAME                                  READY   STATUS    RESTARTS   AGE
pod/app-deployment-85489cdd5b-7294b   1/1     Running   0          14s
pod/app-deployment-85489cdd5b-926jt   1/1     Running   0          14s
pod/app-deployment-85489cdd5b-wjq2b   1/1     Running   0          14s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   44d

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-deployment   3/3     3            3           14s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/app-deployment-85489cdd5b   3         3         3       14s
```
The deployment definition file, will automatically create the replicaset and the pods.

```yaml
# deployment-definition.yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: app-deployment
spec:
  template:
    metadata:
      name: pod-nginx
      labels:
        type: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: frontend
```

```bash
# This explain command will be very handy:
$ kubectl explain deployments

GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
    
FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind  <string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata      <ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec  <DeploymentSpec>
    Specification of the desired behavior of the Deployment.

  status        <DeploymentStatus>
    Most recently observed status of the Deployment.
```


#### Deployment Updates, Rollout, and Versioning

Commands you to run Rollout strategy: 
[1] Rolling update 
[2] Recreate

```bash
$ kubectl create -f deployment-definition.yml
$ kubectl get deployments
$ kubectl apply -f deployment-definition.yml
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
$ kubectl rollout status deployment/myapp-deployment
$ kubectl rollout history deployment/myapp-deployment
$ kubectl rollout undo deployment/myapp-deployment
$ kubectl create -f deployment-definition.yml --record=true
```

```bash
$ kubectl get pods -o wide

NAME                                    READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
app-deployment-nginx-69976db885-5wn4v   1/1     Running   0             13m   10.244.0.34   minikube   <none>           <none>
app-deployment-nginx-69976db885-7q9qx   1/1     Running   0             13m   10.244.0.36   minikube   <none>           <none>
app-deployment-nginx-69976db885-xv58d   1/1     Running   0             13m   10.244.0.35   minikube   <none>           <none>
hello-node-7b4b746b66-9cr8x             1/1     Running   3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

$ kubectl get all -o wide 

NAME                                        READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-69976db885-5wn4v   1/1     Running   0             13m   10.244.0.34   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-7q9qx   1/1     Running   0             13m   10.244.0.36   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-xv58d   1/1     Running   0             13m   10.244.0.35   minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running   3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node                    LoadBalancer   10.103.22.214    <pending>     8080:31963/TCP   45d   k8s-app=hello-node
service/kubernetes                    ClusterIP      10.96.0.1        <none>        443/TCP          45d   <none>

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                         SELECTOR
deployment.apps/app-deployment-nginx   3/3     3            3           13m   nginx-container   nginx                                          type=frontend
deployment.apps/hello-node             1/1     1            1           45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node

NAME                                              DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                         SELECTOR
replicaset.apps/app-deployment-nginx-69976db885   3         3         3       13m   nginx-container   nginx                                          pod-template-hash=69976db885,type=frontend
replicaset.apps/hello-node-7b4b746b66             1         1         1       45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node,pod-template-hash=7b4b746b66


$ kubectl edit deployment app-deployment-nginx
deployment.apps/app-deployment-nginx edited

$ kubectl rollout status deployment app-deployment-nginx
deployment "app-deployment-nginx" successfully rolled out

$ kubectl rollout history deployment app-deployment-nginx
deployment.apps/app-deployment-nginx 
REVISION  CHANGE-CAUSE
1         <none>

$ kubectl get all -o wide

NAME                                        READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-69976db885-5wn4v   1/1     Running   0             18m   10.244.0.34   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-7q9qx   1/1     Running   0             18m   10.244.0.36   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-xv58d   1/1     Running   0             18m   10.244.0.35   minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running   3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node                    LoadBalancer   10.103.22.214    <pending>     8080:31963/TCP   45d   k8s-app=hello-node
service/kubernetes                    ClusterIP      10.96.0.1        <none>        443/TCP          45d   <none>

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                         SELECTOR
deployment.apps/app-deployment-nginx   3/3     3            3           18m   nginx-container   nginx                                          type=frontend
deployment.apps/hello-node             1/1     1            1           45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node

NAME                                              DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                         SELECTOR
replicaset.apps/app-deployment-nginx-69976db885   3         3         3       18m   nginx-container   nginx                                          pod-template-hash=69976db885,type=frontend
replicaset.apps/hello-node-7b4b746b66             1         1         1       45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node,pod-template-hash=7b4b746b66

----- ERROR -----

$ kubectl set image deployment nginx-container=nginx:1.18
error: resource(s) were provided, but no name was specified

----- ERROR -----

$ kubectl set image deployment app-deployment-nginx nginx-container=nginx:1.18
deployment.apps/app-deployment-nginx image updated

$ kubectl rollout status deployment app-deployment-nginx  
Waiting for deployment "app-deployment-nginx" rollout to finish: 0 of 3 updated replicas are available...
Waiting for deployment "app-deployment-nginx" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "app-deployment-nginx" rollout to finish: 2 of 3 updated replicas are available...
deployment "app-deployment-nginx" successfully rolled out

$ kubectl get all -o wide 

NAME                                        READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-7554f76c4c-6st79   1/1     Running   0             37s   10.244.0.42   minikube   <none>           <none>
pod/app-deployment-nginx-7554f76c4c-gs9cn   1/1     Running   0             37s   10.244.0.40   minikube   <none>           <none>
pod/app-deployment-nginx-7554f76c4c-pnchj   1/1     Running   0             37s   10.244.0.41   minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running   3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node                    LoadBalancer   10.103.22.214    <pending>     8080:31963/TCP   45d   k8s-app=hello-node
service/kubernetes                    ClusterIP      10.96.0.1        <none>        443/TCP          45d   <none>

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                         SELECTOR
deployment.apps/app-deployment-nginx   3/3     3            3           20m   nginx-container   nginx:1.18                                     type=frontend
deployment.apps/hello-node             1/1     1            1           45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node

NAME                                              DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                         SELECTOR
replicaset.apps/app-deployment-nginx-69976db885   0         0         0       20m   nginx-container   nginx                                          pod-template-hash=69976db885,type=frontend
replicaset.apps/app-deployment-nginx-7554f76c4c   3         3         3       37s   nginx-container   nginx:1.18                                     pod-template-hash=7554f76c4c,type=frontend
replicaset.apps/hello-node-7b4b746b66             1         1         1       45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node,pod-template-hash=7b4b746b66


$ kubectl rollout undo deployment app-deployment-nginx
deployment.apps/app-deployment-nginx rolled back

$ kubectl get all -o wide 

NAME                                        READY   STATUS              RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-69976db885-8qzx9   0/1     ContainerCreating   0             3s    <none>        minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-fsp52   0/1     ContainerCreating   0             3s    <none>        minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-jpc65   0/1     ContainerCreating   0             3s    <none>        minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running             3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node                    LoadBalancer   10.103.22.214    <pending>     8080:31963/TCP   45d   k8s-app=hello-node
service/kubernetes                    ClusterIP      10.96.0.1        <none>        443/TCP          45d   <none>

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                         SELECTOR
deployment.apps/app-deployment-nginx   0/3     3            0           20m   nginx-container   nginx                                          type=frontend
deployment.apps/hello-node             1/1     1            1           45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node

NAME                                              DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                         SELECTOR
replicaset.apps/app-deployment-nginx-69976db885   3         3         0       20m   nginx-container   nginx                                          pod-template-hash=69976db885,type=frontend
replicaset.apps/app-deployment-nginx-7554f76c4c   0         0         0       64s   nginx-container   nginx:1.18                                     pod-template-hash=7554f76c4c,type=frontend
replicaset.apps/hello-node-7b4b746b66             1         1         1       45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node,pod-template-hash=7b4b746b66


$ kubectl get all -o wide

NAME                                        READY   STATUS              RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-69976db885-8qzx9   0/1     ContainerCreating   0             9s    <none>        minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-fsp52   1/1     Running             0             9s    10.244.0.44   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-jpc65   1/1     Running             0             9s    10.244.0.43   minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running             3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node                    LoadBalancer   10.103.22.214    <pending>     8080:31963/TCP   45d   k8s-app=hello-node
service/kubernetes                    ClusterIP      10.96.0.1        <none>        443/TCP          45d   <none>

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                         SELECTOR
deployment.apps/app-deployment-nginx   2/3     3            2           20m   nginx-container   nginx                                          type=frontend
deployment.apps/hello-node             1/1     1            1           45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node

NAME                                              DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                         SELECTOR
replicaset.apps/app-deployment-nginx-69976db885   3         3         2       20m   nginx-container   nginx                                          pod-template-hash=69976db885,type=frontend
replicaset.apps/app-deployment-nginx-7554f76c4c   0         0         0       70s   nginx-container   nginx:1.18                                     pod-template-hash=7554f76c4c,type=frontend
replicaset.apps/hello-node-7b4b746b66             1         1         1       45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node,pod-template-hash=7b4b746b66


$ kubectl get all -o wide

NAME                                        READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-69976db885-8qzx9   1/1     Running   0             11s   10.244.0.45   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-fsp52   1/1     Running   0             11s   10.244.0.44   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-jpc65   1/1     Running   0             11s   10.244.0.43   minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running   3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node                    LoadBalancer   10.103.22.214    <pending>     8080:31963/TCP   45d   k8s-app=hello-node
service/kubernetes                    ClusterIP      10.96.0.1        <none>        443/TCP          45d   <none>

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                         SELECTOR
deployment.apps/app-deployment-nginx   3/3     3            3           20m   nginx-container   nginx                                          type=frontend
deployment.apps/hello-node             1/1     1            1           45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node

NAME                                              DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                         SELECTOR
replicaset.apps/app-deployment-nginx-69976db885   3         3         3       20m   nginx-container   nginx                                          pod-template-hash=69976db885,type=frontend
replicaset.apps/app-deployment-nginx-7554f76c4c   0         0         0       72s   nginx-container   nginx:1.18                                     pod-template-hash=7554f76c4c,type=frontend
replicaset.apps/hello-node-7b4b746b66             1         1         1       45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node,pod-template-hash=7b4b746b66


$ kubectl get all -o wide

NAME                                        READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-69976db885-8qzx9   1/1     Running   0             14s   10.244.0.45   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-fsp52   1/1     Running   0             14s   10.244.0.44   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-jpc65   1/1     Running   0             14s   10.244.0.43   minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running   3 (12h ago)   45d   10.244.0.23   minikube   <none>           <none>

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node                    LoadBalancer   10.103.22.214    <pending>     8080:31963/TCP   45d   k8s-app=hello-node
service/kubernetes                    ClusterIP      10.96.0.1        <none>        443/TCP          45d   <none>

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                         SELECTOR
deployment.apps/app-deployment-nginx   3/3     3            3           20m   nginx-container   nginx                                          type=frontend
deployment.apps/hello-node             1/1     1            1           45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node

NAME                                              DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                         SELECTOR
replicaset.apps/app-deployment-nginx-69976db885   3         3         3       20m   nginx-container   nginx                                          pod-template-hash=69976db885,type=frontend
replicaset.apps/app-deployment-nginx-7554f76c4c   0         0         0       75s   nginx-container   nginx:1.18                                     pod-template-hash=7554f76c4c,type=frontend
replicaset.apps/hello-node-7b4b746b66             1         1         1       45d   hello-node        registry.k8s.io/e2e-test-images/agnhost:2.39   k8s-app=hello-node,pod-template-hash=7b4b746b66

```

---
## :rotating_light: Certification Tip:

Here’s a tip!

As you might have seen already, creating and editing YAML files is a bit difficult, especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from the browser to the terminal. Using the kubectl run command can help in generating a YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with a specific name and image, you can simply run the kubectl run command.

Use the below set of commands and try the previous practice tests again, but this time, try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises.

Reference (Bookmark this page for the exam. It will be very handy):

https://kubernetes.io/docs/reference/kubectl/conventions/

Create an NGINX Pod

```
$ kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)

```
$ kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Create a deployment

```
$ kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)

```
$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

```
$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

```
$ kubectl create -f nginx-deployment.yaml
```
OR

In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.

```
$ kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

----

### Services

[1] Services enable connectivity with the internal components and external components.
[2] Services can work with the applications internally.

There are three different types of Services:

- NodePort - Range - 30000 to 32767 
- ClusterIP
- LoadBalancer

#### NodePort Service

```bash
+-----------------------------------------------------------+
|                       K8s Architecture                    |
+-----------------------------------------------------------+
|                          Minikube                         |
|-----------------------------------------------------------|
| +-----------------------------------------------------+   |
| |                 Minikube VM                         |   |
| |-----------------------------------------------------|   |
| | Node IP: 192.168.49.2                               |   |
| | Pod CIDR: 10.244.0.0/24                             |   |
| |                                                     |   |
| |                                                     |   |
| |                                                     |   |
| |                                                     |   |
| |      | Port 30007  |                                |   |
| | +--------------------+                              |   |
| | | NodePort Service   |                              |   |
| | | IP: 10.244.0.43    |                              |   |
| | | app-deployment-nginx                              |   |
| | +--------------------+                              |   | 
| |       | Port 80  |                                  |   |
| |         |      |                                    |   |
| |         |      |                                    |   |
| |         |      |                                    |   |
| |         |      |                                    |   | 
| |       | Port 80  |                                  |   |
| | +--------------------+   +-----------------------+  |   |
| | | Pod A              |   | Pod B                 |  |   |
| | | IP: 10.244.0.43    |   | IP: 10.244.0.44       |  |   |
| | | app-deployment-nginx   |                       |  |   |
| | +--------------------+   +-----------------------+  |   |
| | +--------------------+   +-----------------------+  |   |
| | | Pod C              |   | Pod D                 |  |   |
| | | IP: 10.244.0.45    |   | IP: 10.244.0.23       |  |   |
| | |                    |   | hello-node            |  |   |
| | +--------------------+   +-----------------------+  |   |
| |                              | Port 3232  |         |   |
| |                                                     |   |
| +-----------------------------------------------------+   |
|                                                           |
+-----------------------------------------------------------+
```

Now, when I curl from outside the VM i.e. from the internet, I should be able to get access to the pods/containers/applications inside the pods. In this scenario, it is `NGINX` application within the pods A, B, and C.

> [!NOTE]  
> When I try to access the website URL: `192.168.49.2:30007`, I should be able to reach the nginx pod exposed via the `service port 80`.
> Also, the NodePort service will automatically route the traffic between the pods within the same node or within the different nodes too. Basically, it act as a LoadBalancer and there is no need to specify or deploy it manually.

#### Example:

- Before creating the service:


```bash
$ kubectl get all -o wide   

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node   LoadBalancer   10.103.22.214   <pending>     8080:31963/TCP   46d   k8s-app=hello-node
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          46d   <none>
```

- Post creating the service using the below definition file:

```yaml
# service-definition.yaml
apiVersion: v1
kind: Service
metadata: 
  name: nginx-service
  labels:
    type: nodeport-service
spec:
  type: NodePort
  ports:
    - port: 80                   # service port that connects as a tunnel to the targetPort and the nodePort
      targetPort: 80             # nginx pod port which called as targetPort
      nodePort: 30007            # port that is in the node that gets exposed to the outside world or internet
  selector:
    app: prod-app
```

```bash
$ kubectl create -f service-definition.yaml 
service/nginx-service created
```

```bash
$ kubectl get services -o wide
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
hello-node      LoadBalancer   10.103.22.214   <pending>     8080:31963/TCP   46d    k8s-app=hello-node
kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP          46d    <none>
nginx-service   NodePort       10.102.73.198   <none>        80:30007/TCP     2m5s   app=prod-app
```

You can also use `minikube service <service-name> --url` to get the URL that you can connect.

#### ClusterIP Service & LoadBalancer Service

- ClusterIP connects the internal pods and does not have any exposed ports to the internet.

- LoadBalancer, it is basically the `NodePort` but it works well when using in the `GCP`, `AWS`, and `Azure` environment where it deploys the `LoadBalancer` in between the pods and the node(s).

```yaml
# clusterip-service-definition.yaml
apiVersion: v1
kind: Service
metadata: 
  name: nginx-service
  labels:
    type: clusterIP-service
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: prod-app
```
```yaml
# loadbalancer-service-definition.yaml
apiVersion: v1
kind: Service
metadata: 
  name: nginx-service
  labels:
    type: nodeport-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
  selector:
    app: prod-app
```
when created the `service`, you can check the complete details using the `describe` service command:

```bash
kubectl describe service <service_name>

OR

kubectl describe svc <service_name>
```

> [!NOTE]
> Make use of the imperative commands i.e. the shortcut commands used in K8s from the [kubernetes official documentation](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/).


----

### Namespaces

There are three important namespaces:

- kube-system (where, the k8s components are isolated)
- default
- kube-public (this is where the resources made available to the users)

Namespaces are mostly used in the `dev` and `prod` environments.

Each of the namespaces can have the own `resource-limits` and `policies` so that you can instruct them to do what.

To create namespace:

```yaml
# namespace-dev.yaml
apiVersion: v1
kind: Namespace
metadata:
    name: dev   
```

Followed by:

```bash
$ kubectl create -f namespace-dev.yaml
```

Or, using the `implicit` commands:

```bash
$ kubectl create namespace dev
```

#### Additional namespace commands:

```bash
# Get pods from a specific namespace
$ kubectl get pods --namespace=dev

# Get pods from the default namespace
$ kubectl get pods 

# To set a specific namespace permanently instead of default namespace
$ kubectl config set-context $(kubetl config current-context) --namespace=dev

# To get the pods in all namespaces
$ kubectl get pods --all-namespaces
```

#### How to Add Resource Limits to the Namespaces

```yaml
# compute-quota.yaml

apiVersion: v1
kind: ResourceQuota
metadata: 
    name: compute-quota
    namespace: dev
spec:
    hard:
        pods: "10"
        requests.cpu: "4"
        requests.memory: 5Gi
        limits.cpu: "10"
        limits.memory: 10Gi
```

```bash
$ kubectl create -f compute-quota.yaml
```
Get all namespaces

```bash
$ kubectl get namespaces

NAME              STATUS   AGE
default           Active   8m45s
dev               Active   30s
finance           Active   30s
kube-node-lease   Active   8m45s
kube-public       Active   8m45s
kube-system       Active   8m46s
manufacturing     Active   30s
marketing         Active   30s
prod              Active   30s
research          Active   30s
```

```bash
$ kubectl get pods --all-namespaces

OR

$ kubectl get pods -A
```

### Imperative and Declarative Commands

There are different approaches to manage IaaC - Infrastructure as a code which are classified as `Imperative` and `Declarative`.

#### Imperative Commands:
```bash
## Creating Objects
$ kubectl run --image=nginx nginx
$ kubectl create deployment --image=nginx nginx
$ kubecti expose deployment nginx --port 80

## Updating Objects
$ kubectl edit deployment nginx
$ kubectl scale deployment nginx --replicas=5
$ kubectl set image deployment nginx nginx=nginx:1.18

# Imperative Object Configuration Files
## Creating/Editing(replace)/Deleting Objects 
$ kubectl create -f nginx. yaml
$ kubectl replace -f nginx.yaml
$ kubectl delete -f nginx.yaml
```

Imperative commands are a quick commands that are limited and cannot run in complex environments.

Creating Manifests file, configuration files will give us and an other user to understand what is configured and how the containers or pods or deployments are running.

> [!IMPORTANT]
> When using the `$ kubectl edit pod nginx` will only edit the pod and the changes are `NOT RECORDED` 
> To RECORD the changes, we should use `$ kubectl replace -f nginx.yaml`, only after making changes to the `nginx.yaml` file.
> There are times when you wanted to completely delete the existing objects and create a new objects so you can use `$ kubectl replace --force -f nginx.yaml`.

It throws errors when try to perform any operations and it fails.

From an exam perspective, it makes you faster. If you wanted to `edit` a deployment, you can run it using the edit command.

#### Declarative 

Using the same configuration file, you can run the declarative way:

```bash
## Create Objects
$ kubectl apply -f nginx.yaml
```

```bash
## Update Objects
$ kubectl apply -f nginx.yaml
```

For an exam perspective, if you wanted to create a multi-pod architecture, then `Declarative` is the best way.

----
## :rotating_light: Certification Tip:

While you would be working mostly the declarative way – using definition files, imperative commands can help in getting one-time tasks done quickly, as well as generate a definition template easily. This would help save a considerable amount of time during your exams.

Before we begin, familiarize yourself with the two options that can come in handy while working with the below commands:

`--dry-run`

 By default as soon as the command is run, the resource will be created. If you simply want to test your command, use the `--dry-run=client` option. This will not create the resource; instead, it tells you whether the resource can be created and if your command is right.

`-o yaml`

 This will output the resource definition in YAML format on the screen.

Use the above two in combination to generate a resource definition file quickly that you can then modify and create resources as required instead of creating the files from scratch.

#### POD

Create an NGINX Pod
```bash
$ kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)

```bash
$ kubectl run nginx --image=nginx --dry-run=client -o yaml
```

#### Deployment

Create a deployment

```bash
$ kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)

```bash
$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```
Generate Deployment with 4 Replicas

```bash
$ kubectl create deployment nginx --image=nginx --replicas=4
```
You can also scale a deployment using the scale command:
```bash
$ kubectl scale
```
```bash
$ kubectl scale deployment nginx--replicas=4
```
Another way to do this `is to save the YAML` definition to a file and modify

```bash
$ kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```
You can then update the YAML file with the replicas or any other field before creating the deployment.

#### Service

Create a Service named `redis-service` of type `ClusterIP` to expose pod `redis` on port `6379`
```bash
$ kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

**_(This will automatically use the pod’s labels as selectors)_**

Or

```bash
$ kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
```
**_(This will not use the pods labels as selectors, instead, it will assume selectors as app=redis. You cannot pass in selectors as an option. So, it does not work very well if your pod has a different label set. So, generate the file and modify the selectors before creating the service)_**

Create a Service named `nginx` of type `NodePort` to expose pod nginx’s port `80` on port `30080` on the nodes:

```bash
$ kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```
**_(This will automatically use the pod’s labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port manually before creating the service with the pod.)_**

Or

```bash
$ kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```
**_(This will not use the pod labels as selectors.)_**

Both the above commands have their own challenges. While one of them cannot accept a selector, the other cannot accept a node port. I would recommend going with the

```bash
$ kubectl expose
```
command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

Reference:
> Kubectl Commands: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
> Conventions: https://kubernetes.io/docs/reference/kubectl/conventions/

-----

### Kubectl Apply

```bash
kubectl apply -f nginx.yaml
```

There are three states for this `apply` command:

- Local file
- Live Object Configuration
- Last Applied Configuration

`Live Object configuration`, when the object is not available, it creates with the system fields.

`Last applied configuration` - it's basically to understand what fields were removed.

Here you go, refer to this documentation - [Merging changes to primitive fields 
](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/#merging-changes-to-primitive-fields)


Where is this `Last applied configuration` stored? It is stored in the `annotions` under the `metadata` property which names `last-applied-configuration`.

From kodekloudhub: 

We have created a repository with notes, links to documentation, and answers to practice questions here. Please make sure to go through these as you progress through the course:

https://github.com/kodekloudhub/certified-kubernetes-administrator-course

----

### Scheduling Section Introduction

#### Manual Scheduling

Different way to Manually scheduling a pod on a node.


#### How scheduling works?

Usually `nodeName` field is not set in the `pod-definition.yaml`, but k8s adds it automatically. This is important to `Bind` the `pod`.

[1] The scheduler goes through all the pods and identifies all the pods where the `nodeName` parameter is NOT SET. Those are the candidates for scheduling.
[2] Once identifies those pods, it schedules the pod to a Node. By setting the `nodeName` to that pod.
[3] Basically, it is called `Binding` where, it binds the pod to a node by creating a `binding object`.


If there are `no scheduler` or if you do not want to use the `built-in scheduler` and you want allocate the pods to the nodes, then we need to manually bind the pod to the nodes.

Also, if the pods are NOT scheduled to a specific state, the pods continues to be in a `PENDING` state.

Without a scheduler, the easiest way to schedule the pod to a node is to add the `nodeName` field/parameter/property to the `pod-definition.yaml` file while creating the pod itself.

> [!Note] You cannot modify or add the `nodeName` field/parameter/property when the pod is already created.

#### What if the pod is already created and assign the pod to a NODE?

A way to create assign a node to an existing pod is to create a binding object `pod-bind-definition.yaml` and in the binding object, you point to a target node parameter called `target.kind`, `target.name`, and `target.apiVersion` and then run a post request.

Example scenario: 

An `nginx` pod is already created and we need to add a `binding object` to allocate the pod to a Node `node02`.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
    name: nginx
    labels:
        app: nginx
spec:
    containers:
        - image: nginx
          name: nginx
          ports:
            - containerPort: 80
```

```yaml
# pod-bind-definition.yaml

apiVersion: v1
kind: Binding
metadata:
    name: nginx
target:
    apiVersion: v1
    kind: Node
    name: node02
```

Now, finally run a `POST` request:

```bash
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind":"Binding", ...}' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
```

> [!IMPORTANT] 
> How will you run the above `curl` command?
> basically, the `$SERVER` is `kube-proxy` so you can start the server by running `kubectl proxy` in a separate terminal so that it gives you a SERVER Endpoint
> like `127.0.0.1:8001`
> And, for `$PODNAME`, you can add it as an environmental variable or, can add the podname directly.
> Refer to this [K8s official documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/#:~:text=new%20terminal%2C%20run%3A-,kubectl%20proxy,-Now%20again%2C%20we%27ll)
> 
> Create a `pod-binding.json` file:
> ```json
> {
>  "apiVersion": "v1",
>  "kind": "Binding",
>  "metadata": {
>    "name": "nginx"
> },
>  "target": {
>    "apiVersion": "v1",
>    "kind": "Node",
>    "name": "controlplane"
> }
> }
>
> # Please note, you would need to pass the JSON format as above and NO Flattened JSON format is accepted or recognized.
>
> ```
> In the end, the request looks like this:
> ```
> curl --header "Content-Type:application/json" --request POST --data @pod-binding.json http://127.0.0.1:8001/api/v1/namespaces/default/pods/nginx/binding/
> ```
> 
> ```json
> # Output looks like this:
> {
>   "kind": "Status",
>   "apiVersion": "v1",
>   "metadata": {},
>   "status": "Success",
>   "code": 201
> }
> ```

----

### Labels and Selectors

#### What is Labels and Selectors in General in K8s?

It is standard method to group things together i.e. pods, replicasets, services as such. It is basically filtering the specific pods in midst of 100s of pods. 

Labels are the properties that are attached to an object and the Selectors are the one that selects i.e. it filters among other objects.

To use `Selector` to filter the pods(can filter other objects too):

```bash
$ kubectl get pods --selector app=App1

$ kubectl get pods --selector env=prod,bu=finance,tier=frontend
```

> [!TIP] As there could be many pods, you can count the pods using the `wordcount` command i.e. `wc`. Also, use `--no-headers` to remove the headers.
> ```bash
> kubectl get pods --selector env=dev --no-headers | wc -l
> ```

####  Annotations

It records the details of the replicaset like the `buildversion`, `last_configured_options` and so on.

---

### Taints and Tolerations

Concepts of Taints and Tolerations

- Taints are set on Nodes
- Tolerations are set on Pods

#### How can we `Taint` a Node?

```bash
$ kubectl taint nodes node-name key=value:taint-effect
```
#### What is `taint-effect`?

It is the option that we tell the Node that `What happens to PODs that do not tolerate this taint`.

There are three options of `taint-effect`:
* NoSchedule
* PreferNoSchedule
* NoExecute

- NoExecute:
This affects pods that are already running on the node as follows:
Pods that do not tolerate the taint are evicted immediately
Pods that tolerate the taint without specifying tolerationSeconds in their toleration specification remain bound forever
Pods that tolerate the taint with a specified tolerationSeconds remain bound for the specified amount of time. After that time elapses, the node lifecycle controller evicts the Pods from the node.

- NoSchedule:
No new Pods will be scheduled on the tainted node unless they have a matching toleration. Pods currently running on the node are not evicted.

- PreferNoSchedule:
PreferNoSchedule is a "preference" or "soft" version of NoSchedule. The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, but it is not guaranteed.

Please refer to this [Taint-effect in K8s documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#:~:text=The%20allowed%20values,is%20not%20guaranteed.)

```bash
$ kubectl taint nodes node1 app=blue:NoSchedule
```

#### How can we apply `Tolerations` to a Pod?

To add a Toleration to a pod:

```yaml
#pod-definition.yaml

apiVersion: v1
kind: Pod
metadata: 
    name: nginx
    labels:
        app: blue
spec:
    containers:
        - image: nginx
          name: nginx
          ports:
            - containerPort: 8080
    tolerations:
        - key: "app"
          operator: "Equal"
          value: "blue"
          effect: "NoSchedule"

```

You can check this Taints and Tolerations in a `Minikube` cluster because, in the Minikube cluster, the `Master` node does not have any taints and it is the only one that is available to host the pods so, the `scheduler` assigns the pods to the master node. 

But, in a `K8s cluster`(NOT minikube), you will be able to notice that the `Master` node is protected by a `Taint` because, it strictly tells the `scheduler` to NOT schedule pods on the Master node.

You can use this command to understand where there is any Taint in the `kubemaster`:

```bash
$ kubectl describe node kubemaster | grep Taint
```

To remove the `Taint` from the node, we can use the below command:

```bash
$ kubectl taint node controlplane node-role.kuberentes.io/master:NoSchedule-
```

The minus(-) in the end of the above command removes the taint.

Here is the [official documentation on the Taint removal](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#:~:text=To%20remove%20the%20taint%20added%20by%20the%20command%20above%2C%20you%20can%20run%3A)

----

### Node Selectors

Let's start with `Label the Nodes`, without labelling the nodes it is NOT possible to have the `nodeSelector` in the `pod-definition.yaml` file.

```bash
$ kubectl label nodes <node-name> <label-key>=<label-value>

$ kubectl label nodes node-1 size=large
```
Please note, if there are existing labels in the node, the new label(s) that you add will be appended to existing labels.

Now, we can label the node in the pod-definition.yaml file.

```yaml
apiVersion: v1
kind: Pod
metadata: 
    name: nginx
    labels:
        app: nodejs
        env: dev
spec:
    containers:
        - image: nginx
          name: nginx
    nodeSelector:
          size: Large       ## nodeSelector Label that we named in the node above.
```

#### Limitation of using the Node Selector property:

- Can use only label i.e. single label. You cannot give any combinations like 
    `can you select a Medium node or Large node, if the Large node is unavailable`
    `do not select the small node`
- Basically, it cannot perform advanced operations or conditions like `OR` or `NOT` operator.

To overcome this Limitations, the `nodeAffinity` and the `antiAffinity` was developed.

----

### Node Affinity

It provides advanced configuration and capabilities to place pods in the nodes that we want and more flexible but, this is complex as it looks.

```yaml
# Example correlating the above nodeSelector: size - Large

apiVersion: v1
kind: Pod
metadata: 
    name: nginx
    labels:
        app: nodejs
        env: dev
spec:
    containers:
        - image: nginx
          name: nginx
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
              - key: size
                operator: In        # NotIn or Exists(does not compare the values)
                values:
                - Large
                - Medium
```
Adding the same thing in `json` to cross-check:

```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx",
    "labels": {
      "app": "nodejs",
      "env": "dev"
    }
  },
  "spec": {
    "containers": [
      {
        "image": "nginx",
        "name": "nginx"
      }
    ],
    "affinity": {
      "nodeAffinity": {
        "requiredDuringSchedulingIgnoredDuringExecution": {
          "nodeSelectorTerms": [
            {
              "matchExpressions": [{"key": "size","operator": "In","value": ["Large","Medium"]}]
            }
          ]
        }
      }
    }
  }
}
```
```yaml
# Example from the K8s documentation
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```

You can find the above node-affinity.yaml file in the [official K8s documentation.](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#:~:text=To%20remove%20the%20taint%20added%20by%20the%20command%20above%2C%20you%20can%20run%3A)

#### Node Affinity Types

This Node Affinity types defines the behaviour of the scheduler and checks during the lifecycle of the pod.

There are two types of Node Affinity available:

- `requiredDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`

There are Two states of the lifecycle of the pods:

- During scheduling
- During execution

- During scheduling

There are just two usecases:

* If the Node Affinity is set to `requiredDuringSchedulingIgnoredDuringExecution`, 

And, the label is `NOT` set to the nodes, then the pod will be stuck in the `Pending` state without scheduling the pods to a specific node as all the nodes does not have a label.

But, if a label is set to a node, then the pod will be placed accordingly.

* If the Node Affinity is set to `preferredDuringSchedulingIgnoredDuringExecution`,

If the label is `NOT` set to the nodes, then the pod will still be placed on the nodes that does not have the labels, as it is `preferred` and NOT `required` which makes it mandatory to schedule. This will place the pod on any available nodes.

- During Execution:

If the `label` is removed from the node once the pods are placed or while running, the pods will continue to run without getting impacted by this change.

To address this `feature gap`, the `K8s` community is working two more `Node Affinity types`, that are:

- `requiredDuringSchedulingRequiredDuringExecution`
- `preferredDuringSchedulingRequiredDuringExecution`

So basically, this addresses during the pod during the `Execution` as well.

## Example exercise - Practice Test - Node Affinity:
 
```yaml
## $ cat deployment_red.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  selector:
    matchLabels:
      name: blue
  replicas: 3
  template:
    metadata:
      labels:
        name: blue
    spec:
      containers:
        - name: nginx-container
          image: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In        # NotIn or Exists(does not compare the values)
                  values:
                  - blue
                  
```
To quickly edit the deployment definition file, we can use `vi` editor.

```bash
# To edit multiple occurences of a single word:

:%s/old_word/new_word/g

# Taking for example, let's we need to change the string `blue` in the above deployment-definition.yaml file to `red`

:%s/blue/red/g
```
-----

### Resource Limits

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    type: frontend
spec:
  containers:
    - name: nginx
      image: nginx
      ports: 
      - containerPort: 8080
      resources:
        requests:
          memory: "1Gi"
          cpu: 1
        limits:
          memory: "2Gi"
          cpu: 2
```

OOM Limit hit: Out-of-Memory

Default Behavior:


CPU Behavior:
No Requests and No Limits

No Requests and Have Limits

Have Limits and Have Requests

Have Requests and No Limits - This is most ideal setup that allows the other pods to use the vCPUs.

Memory Behavior:

No Requests and No Limits

No Requests and Have Limits

Have Limits and Have Requests

Have Requests and No Limits - This is most ideal setup that allows the other pods to use the memory.

#### LimitRange

[LimitRange K8s Doc](https://kubernetes.io/docs/concepts/policy/limit-range/)

##### For CPU
```yaml
# limit-range-cpu.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 500m
    max: 
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

##### For Memory

```yaml
# limit-range-memory.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max: 
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```
#### Resource Quota

[Resource Quota K8s Official Doc](https://kubernetes.io/docs/concepts/policy/resource-quotas/)


This is the best method to follow and allocate the right set of resources to all the pods.

```yaml
# resource-quota.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests:
      cpu: 4
      memroy: 4Gi
    limits:
      cpu: 10
      memory: 10Gi
```

------

## Important Tip

#### To Edit a POD
Remember, you CANNOT edit specifications of an existing POD other than the below.

```bash
spec.containers[*].image
spec.initContainers[*].image
spec.activeDeadlineSeconds
spec.tolerations
```
For example, you cannot edit the environment variables, service accounts, and resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the `kubectl edit pod`  command. This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.

![vi_editing_pod_definition](vi_editing_pod_definition.png)
![unable_to_save_pod](unable_to_save_pod.png)

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:
```bash
$ kubectl delete pod webapp
```
Then create a new pod with your changes using the temporary file

```bash
$ kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
```

2. The second option is to extract the pod definition in YAML format to a file using the command
```bash
$ kubectl get pod webapp -o yaml >&nbsp;my-new-pod.yaml
```
Then make the changes to the exported file using an editor (vi editor). Save the changes
```bash
$ vi my-new-pod.yaml
```
Then delete the existing pod
```bash
$ kubectl delete pod webapp
```
Then create a new pod with the edited file
```bash
$ kubectl create -f my-new-pod.yaml
```
Edit Deployments

With Deployments, you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification, with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command
```bash
$ kubectl edit deployment my-deployment
```
-----