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

```bash
# To install `etcdctl`:

apt-get install etcd-client
```
So, now the `etcdctl` can be accessed.

For example:

```bash
$ ./etcdctl set key1 value1 -- works v2.0
$ ./etcdctl get key1 -- works v2.0
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

### Daemon Sets


Creating a `DaemonSet-definition.yaml` is same as `Replicaset-definition.yaml`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```
```bash
$ kubectl create -f daemon-set-definition.yaml
```
To view DaemonSets

```bash
$ kubectl get daemonsets
```

To describe DaemonSets

```bash
$ kubectl describe daemonsets monitoring-daemon
```

#### How does it work?

How does the daemonsets schedules pods in each node correctly? 

Until `v1.12`, the daemonsets were scheduled on the nodes using the node Labels but, post K8s `v1.12`, the daemonsets started using the `nodeAffinity` so that it gets scheduled each nodes appropriately without overlapping with eachother.

----

### Static Pods

You have only kubelet and the containerD runtime interface

```
/etc/kubernetes/manifests
```

It could be any directory path, that you can set the `kubelet` to take but, you would need to pass that PATH before the `kubelet` service is deployed.


```bash
# kubelet.service

ExecStart=/usr/local/bin/kubelet \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --pod-manifest-path=/etc/kubernetes/manifests \\  <-------------> directory path
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
```

Instead of directly providing the path to the directory in the service file, we can also provide a `config` option and point to a `config.yaml` file.

```bash
# kubelet.service

ExecStart=/usr/local/bin/kubelet \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --config=kubeconfig.yaml  \\            <-----------------> This is where you place the definition files in the static pods
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
```

And, then:

```bash
# kubeconfig.yaml

staticPodPath: /etc/kubernetes/manifests
```

#### Why one would want to use Static Pods?

The static pods are created because it does not need other component involvements except the `kubelet` itself. It's very straightforward, once you create a static pod using the directory where the `manifests` file are added, the `kubelet service` will take care of those pods.

#### StaticPods vs DaemonSets

| Static Pods  | DaemonSets |
|---|---|
|  Created by Kubelet | Created by Kube-API server(DaemonSet Controller)  |
| Deploy Control Plane components as Static Pods  | Deploy Monitoring Agents, Logging Agents on nodes  |
| Ignored by Kube-Scheduler  | Ignored by Kube-Scheduler  |


#### How do you find a Static Pods in the midst of other pods?

There are several ways to find the static pods:

- We can check the pod names using the `kubectl get pods` command:

![get pods](get-pods.png)

Where, if you notice in the above image, you can see pods that ends with the name `*-controlplane` so basically, these are static pods because, the static pods gets suffix this in the end.

```bash
# Example: 
* kube-apiserver-controlplane 
* kube-controller-manager-controlplane
* kube-scheduler-controlplane
* etcd-controlplane
```

- Second method, to identify whether the pod is a static pod, we can take a look at the `pod`'s yaml file and can check the property `ownerReferences` where, we can see the `Node` as `kind` instead of the `Pod` or the `ReplicaSet`.

```bash
$ kubectl get pods kube-scheduler-controlplane -n kube-system -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
...
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler-controlplane
  namespace: kube-system
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node          <-------------> Pointing to a Node instead of ReplicaSet or Pod itself
    name: controlplane
    uid: cf893f58-3cbc-4544-be85-863dd43e5818
spec:
...
...
...
```

- **_[Not Recommended Method, but can check]_** - Third method, is to check the `PATH` i.e. `/etc/kubernetes/manifests` whether the `yaml` files are available or no.

#### How do you know what is the PATH of the directory holding the static pod definition files?

You would need to look at the `kubelet` configuration: Where, do you find the `kubelet` configuration file? You need to check the `/var/lib/kubelet/config.yaml` and find for property `staticPodPath` and it's value which is where, you have the static pods definition files.

```yaml
# $ cat  /var/lib/kubelet/config.yaml 
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: cgroupfs
clusterDNS:
- 172.20.0.10
clusterDomain: cluster.local
containerRuntimeEndpoint: ""
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMaximumGCAge: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
    text:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
resolvConf: /run/systemd/resolve/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests      <-------------> StaticPodPath, pointing to the directory where the manifest files are stored.
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```

:rotating_light: 
>[!Important] - Note 1 - The moment you create an place a pod yaml in the above `staticPodPath` i.e. `/etc/kubernetes/manifests`, the pod will be created by the `kubelet` automatically and you don't have to run it manually.

>[!Important] - Note 2 - Please note, the `kubelet` are deployed as a `daemonsets` in each nodes, let it be `controlplane` or `node01` or `N Node...`. So, basically, each `kubelet` can have its own `staticPodPath` and it will NOT be the same for each nodes. So, makesure, you `ssh` each node and understand where the `kubelet- staticPodPath` is pointing to and then, delete or create static pods in that directory.

#### How to quickly create a static Pod?

```bash
$ kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep
 1000 >> static_pod.yaml
```
```
# static_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox
    name: static-busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Place this `static_pod.yaml` in the staticPodPath - `/etc/kubernetes/manifests` and `kubelet` will immediately create a pod.

----

### Multiple Schedulers

We know how the default `kube-scheduler` work, what if you need a separate `scheduler` that have its own conditions.

You can create your `own custom scheduler` and instruct the `pod` or the `deployment` to make use of your `custom scheduler` instead of `default` one.

You can find the `default` scheduler configuration in `scheduler-config.yaml`.

```yaml
# default scheduler
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default-scheduler
```
Creating own schedulers:

```yaml
# custom scheduler 1
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler
```
```yaml
# custom scheduler 2
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler-2
```
#### Deploying Additional Scheduler:

- We'll download the binary from the cloud repo.
- Run, the `kube-scheduler.service` and provide the binary PATH and the config PATH.

```bash
# kube-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler \\    
--config=/etc/kubernetes/config/kube-scheduler.yaml
```
For custom schedulers, we can change the config file:
```bash
# kube-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler \\    <-----------> Make use of the downloaded binary and add the custom scheduler configuration path
--config=/etc/kubernetes/config/my-scheduler-2-config.yaml
```

>[!IMPORTANT] But, this is NOT how you deploy the `custom schedulers` because the `kubeadm` tool deploys `schedulers` as `POD` instead  of a `service`.

#### Deploy Additional Scheduler as a POD

```yaml
# my-custom-scheduler.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --config=/etc/kubernetes/my-scheduler-config.yaml
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

Whereas, in `my-scheduler-config.yaml`

```yaml
# my-scheduler-config.yaml

apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler
leaderElection:                 <-------------> Used when we have multiple master nodes and HA cluster
  leaderElect: true
  resourceNamespace: kube-system
  resourceName: lock-object-my-scheduler
```

[K8s Doc on Multiple Schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

Once, we have created the `custom scheduler`, let's see how we can include it in the `Pod` definition yaml.

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-custom-scheduler
```

#### How to view Events that the right scheduler picked up the POD?

* Option 1:

```bash
$ kubectl get events -o wide
```
```bash
# Output Response
kubectl get events -o wide
LAST SEEN   TYPE      REASON                    OBJECT              SUBOBJECT                SOURCE                                    MESSAGE                                                                                               FIRST SEEN   COUNT   NAME
21m         Normal    Starting                  node/controlplane                            kubelet, controlplane                     Starting kubelet.                                                                                     21m          1       controlplane.1813668e874fc7e1
21m         Warning   CgroupV1                  node/controlplane                            kubelet, controlplane                     Cgroup v1 support is in maintenance mode, please migrate to Cgroup v2.                                21m          1       controlplane.1813668e87679f58
21m         Warning   InvalidDiskCapacity       node/controlplane                            kubelet, controlplane                     invalid capacity 0 on image filesystem                                                                21m          1       controlplane.1813668e87ad31be
21m         Normal    NodeAllocatableEnforced   node/controlplane                            kubelet, controlplane                     Updated Node Allocatable limit across pods                                                            21m          1       controlplane.1813668e8fded882
21m         Normal    NodeHasSufficientMemory   node/controlplane                            kubelet, controlplane                     Node controlplane status is now: NodeHasSufficientMemory                                              21m          1       controlplane.1813668e95fed669
21m         Normal    NodeHasNoDiskPressure     node/controlplane                            kubelet, controlplane                     Node controlplane status is now: NodeHasNoDiskPressure                                                21m          1       controlplane.1813668e95ff05dc
21m         Normal    NodeHasSufficientPID      node/controlplane                            kubelet, controlplane                     Node controlplane status is now: NodeHasSufficientPID                                                 21m          1       controlplane.1813668e95ff19ee
21m         Normal    RegisteredNode            node/controlplane                            node-controller                           Node controlplane event: Registered Node controlplane in Controller                                   21m          1       controlplane.1813668fe11db724
21m         Normal    Starting                  node/controlplane                            kube-proxy, kube-proxy-controlplane                                                                                                             21m          1       controlplane.181366904f93b052
21m         Normal    NodeReady                 node/controlplane                            kubelet, controlplane                     Node controlplane status is now: NodeReady                                                            21m          1       controlplane.1813669079189b92
24s         Normal    Scheduled                 pod/nginx                                    my-scheduler, my-scheduler-my-scheduler   Successfully assigned default/nginx to controlplane                                                   24s          1       nginx.181367bbbc8116e5
23s         Normal    Pulling                   pod/nginx           spec.containers{nginx}   kubelet, controlplane                     Pulling image "nginx"                                                                                 23s          1       nginx.181367bbe355aff4
20s         Normal    Pulled                    pod/nginx           spec.containers{nginx}   kubelet, controlplane                     Successfully pulled image "nginx" in 3.284s (3.284s including waiting). Image size: 72099501 bytes.   20s          1       nginx.181367bca716100c
20s         Normal    Created                   pod/nginx           spec.containers{nginx}   kubelet, controlplane                     Created container nginx                                                                               20s          1       nginx.181367bca92013b3
20s         Normal    Started                   pod/nginx           spec.containers{nginx}   kubelet, controlplane                     Started container nginx                                                                               20s          1       nginx.181367bcb7ba41cf
```

This command will display all the `SOURCE`, `MESSAGE`, and `REASON` where, the pod is scheduled using the specific scheduler or NOT.

* Option 2:

```bash
$ kubectl logs my-custom-scheduler --name-space=kube-system
OR
$ kubectl logs my-custom-scheduler -n kube-system
```

This will give the logs whether the scheduler assigned the pod to the node without any problem.

#### How to create `ConfigMaps`?

```yaml
# $ cat my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
leaderElection:
  leaderElect: false
```

```yaml
# $ cat my-scheduler-configmap.yaml 
apiVersion: v1
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
    leaderElection:
      leaderElect: false
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-scheduler-config
  namespace: kube-system
```

To create the `configMaps`
```bash
$ kubectl create -f my-scheduler-configmap.yaml 
configmap/my-scheduler-config created
```
OR

```bash
$ kubecl create configmap my-scheduler-config --from-file=/root/my-scheduler-configmap.yaml -n kube-system
configmap/my-scheduler-config created
```

```bash
$ kubectl get configmap -A
NAMESPACE         NAME                                                   DATA   AGE
default           kube-root-ca.crt                                       1      15m
kube-flannel      kube-flannel-cfg                                       2      15m
kube-flannel      kube-root-ca.crt                                       1      15m
kube-node-lease   kube-root-ca.crt                                       1      15m
kube-public       cluster-info                                           2      15m
kube-public       kube-root-ca.crt                                       1      15m
kube-system       coredns                                                1      15m
kube-system       extension-apiserver-authentication                     6      15m
kube-system       kube-apiserver-legacy-service-account-token-tracking   1      15m
kube-system       kube-proxy                                             2      15m
kube-system       kube-root-ca.crt                                       1      15m
kube-system       kubeadm-config                                         1      15m
kube-system       kubelet-config                                         1      15m
kube-system       my-scheduler-config                                    1      4s
```

#### How to create a proper scheduler pod definition file?

```yaml
# $ cat my-scheduler.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-scheduler
  name: my-scheduler
  namespace: kube-system
spec:
  serviceAccountName: my-scheduler
  containers:
  - command:
    - /usr/local/bin/kube-scheduler
    - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
    image: <use-correct-image>
    livenessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 15
    name: kube-second-scheduler
    readinessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
    resources:
      requests:
        cpu: '0.1'
    securityContext:
      privileged: false
    volumeMounts:
      - name: config-volume
        mountPath: /etc/kubernetes/my-scheduler
  hostNetwork: false
  hostPID: false
  volumes:
    - name: config-volume
      configMap:
        name: my-scheduler-config
```

#### What are its Limitations of having Multiple Schedulers?

- First and foremost, maintaining multiple schedulers at the same time is very challenging and as it runs as a service, it is difficult for schedulers to know whether the other schedulers are scheduling any pods/workloads to the same node at the same time. This creates a conflict, that's when we need something called as `Scheduler Profiles` to manage it automatically and each schedulers knows what are the neighboring schedulers.
----

### Scheduler Profiles

To understand what is Scheduler Profiles, we need to understand how Scheduling works, the process from the scheduling, choosing the best node via Scoring mechanism, and binding it to a specific node.

The different phases that the pod goes through before it runs within a node:

- Scheduling Queue
- Filtering phase
- Scoring phase
- Binding phase


In among the phases, there are different scheduling plugins:

#### Scheduling Queue:
- PrioritySort - It is reponsible for queueing/moving the pods in the queue that got high priority, compared to the lower priority. So, the high priority pod is executed first.

#### Filtering phase:

- NodeResourcesFit - It is responsible for identifying the nodes that got sufficient resources to `fit the pods`, and filters out those nodes that does not fit the pods.
- NodeName - It is responsible in identifying whether the pod-definition file is provided with `nodeName` parameter and if yes, it filters out other nodes and will align the pod with the node that is mentioned in the pod-definition file.
- NodeUnschedulable - It is responsible to not schedule the pods when the nodes have set the flag `Unschedulable` to `true` i.e. it instructs the scheduler to do NOT schedule the pods in this node where the flag is enabled.

#### Scoring:

- NodeResourcesFit - In this phase, it is responsible to identify the node that has less resources and more resources when the pod is scheduled. It runs a prediction algorithm, assuming that if the pod is scheduled the nodes, then how much resources are remaining. The node that got the highest resource post scheduling the pods, will be considered as a `fit`.
- ImageLocality - Associates a `high score` to the nodes that has already a copy of the `image` and make the pod to get assigned to the node even though, the other node has higher resources when compared.

#### Binding:

- DefaultBinder - It is responsible to bind the pod to the nodes that got scored in the previous phases to a specific node.

>[!Important]
> K8s cluster is highly extensible so, you can write your `own plugins` and add them in these phases using the `EXTENSION` points.
> At each phase, you can attach the `plugin` to an `extension point`.
> For example, the `PrioritySort` plugin from the `Scheduling Queue` phase, can be attached to the `queueSort` extension point. Other extensions are, `preFilter`, `Filter`,`PostFilter`, `PreScore`, `Score`, `Reserve`, `Permit`, `PreBind`, `Bind`, `PostBind` and more.

To gain detailed understanding of `What plugins gets attached to different Extension Points`, refer to this [K8s Official Documentation on Scheduling Plugins](https://kubernetes.io/docs/reference/scheduling/config/#scheduling-plugins)

The extension points can expand across multiple plugins.

#### Let's understand how to modify the default behavior of the plugins are scheduled and how to add our own plugins

Instead of running each schedulers separately, we can add all the schedulers under the same profile so that they know the status of each schedulers while assigning or allocating the pods/workloads to a specific node.

```yml
# my-scheduler-2-config.yaml
apiVersion: kubescheduler.config.k8s.io/vl
kind: KubeSchedulerConfiguration 
profiles:
- schedulerName: my-scheduler-2
- schedulerName: my-scheduler-3
- schedulerName: my-scheduler-4
```
To specify, the schedulers what to do, we can add the configuration below the schedulers itself:
```yaml
# my-scheduler-2-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration 
profiles:
- schedulerName: my-scheduler-2 
plugins:                          <-----------------> Instructing the plugins to disable and enable to use the custom plugins.
  score:
    disabled:
      - name: TaintToleration 
    enabled:
      - name: MyCustomPluginA
      - name: MyCustomPluginB
- schedulerName: my-scheduler-3
  plugins:                        <-----------------> Removing/disabling all the plugins through the extension points here.
    preScore: 
      disabled:
        - name: '*' 
    score:
      disabled:
        - name: '*'
- schedulerName: my-scheduler-4
```

This functionality is made available post `K8s v1.18`. Here is the [official K8s release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#:~:text=Scheduling%20and%20Testing%5D-,Kube%2Dscheduler%20can%20run%20more%20than%20one%20scheduling%20profile.%20Given%20a%20pod%2C%20the%20profile%20is%20selected%20by%20using%20its%20.spec.schedulerName.%20(%2388285%2C%20%40alculquicondor)%20%5BSIG%20Apps%2C%20Scheduling%20and%20Testing%5D,-Scheduler%20Extenders%20can)

#### Very Important Resources for reference regarding the Schedulers and how it works:

- https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduling_code_hierarchy_overview.md

- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/

- https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/

- https://stackoverflow.com/questions/28857993/how-does-kubernetes-scheduler-work

----------

## :: LOGGING AND MONITORING 

----------

### Monitoring Cluster Components

#### How to do monitoring the resource consumption in K8s cluster? in specific, what would you like to monitor? Node level metrics, such as number_of_nodes, and how many of them in the cluster, memory, disk. And, pod level metrics!

However, for now, the K8s does not come up with built-in monitoring solution.

HEAPSTER - One of the initial projects that was built as an monitoring solution. But, now it is deprecated.

As an successor solution of the Heapster, the `Metrics Server` was created. Here, the `Metrics Server` collects the monitoring metrics from the nodes & pods, and aggregates & stores in the `In-Memory` datastore and cannot view the Historical metrics but, can view the current metrics of the pods or nodes.

To monitor and check the Historical Performance metrics, then you would need to make use of solutions like `Elastic Stack`, `Dynatrace`, `Datadog`, `Prometheus` and so on.

#### So, let's understand how are the metrics generated on the pods in a K8s cluster?

As we know, there is a `Kubelet` deployed as `agent` in each nodes, which talks with the `kube-apiserver` and collects the `pod metrics` using a sub-component within the `kubelet` called as `cAdvisor` i.e. `container Advisor` that collects the pod metrics and exposes the metrics via, kubelet > kube-apiserver > Metrics Server.

#### How to deploy Metrics Server in an existing K8s cluster?

In Minikube:

```bash
$ minikube addons enable metrics-server
```

And, while using `kubeadm` or other `K8s solutions`:

Previously used:
```bash
$ git clone https://github.com/kubernetes-incubator/metrics-server.git
```

Latest Metrics Server repo: https://github.com/kubernetes-sigs/metrics-server

```bash
$ git clone https://github.com/kubernetes-sigs/metrics-server.git
```

And, run:

```bash
$ kubectl create -f deploy/1.8+/
```
or, you can directly pull the `components.yaml` from the `Metrics Server` repo and apply it:

```bash
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
```yaml
# components.yaml

$ cat components.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        image: registry.k8s.io/metrics-server/metrics-server:v0.7.2
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 10250
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```

Once, it deploys all the relevant objects/pods/services/ to run the `Metric Server`:
```bash
#command:
$ kubectl apply -f components.yaml 

#output:
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

Can cross-verify with:
```bash
$ kubectl get all -A
NAMESPACE      NAME                                       READY   STATUS    RESTARTS   AGE
default        pod/elephant                               1/1     Running   0          9m1s
default        pod/lion                                   1/1     Running   0          9m1s
default        pod/rabbit                                 1/1     Running   0          9m1s
kube-flannel   pod/kube-flannel-ds-2zpnc                  1/1     Running   0          18m
kube-flannel   pod/kube-flannel-ds-692pz                  1/1     Running   0          18m
kube-system    pod/coredns-77d6fd4654-8xqt4               1/1     Running   0          18m
kube-system    pod/coredns-77d6fd4654-k525p               1/1     Running   0          18m
kube-system    pod/etcd-controlplane                      1/1     Running   0          18m
kube-system    pod/kube-apiserver-controlplane            1/1     Running   0          18m
kube-system    pod/kube-controller-manager-controlplane   1/1     Running   0          18m
kube-system    pod/kube-proxy-msg6g                       1/1     Running   0          18m
kube-system    pod/kube-proxy-pl7ww                       1/1     Running   0          18m
kube-system    pod/kube-scheduler-controlplane            1/1     Running   0          18m
kube-system    pod/metrics-server-54bf7cdd6-k7vfj         1/1     Running   0          3m40s    ------ Metrics Server Pod

NAMESPACE     NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes       ClusterIP   172.20.0.1      <none>        443/TCP                  18m
kube-system   service/kube-dns         ClusterIP   172.20.0.10     <none>        53/UDP,53/TCP,9153/TCP   18m
kube-system   service/metrics-server   ClusterIP   172.20.73.248   <none>        443/TCP                  3m40s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   2         2         2       2            2           <none>                   18m
kube-system    daemonset.apps/kube-proxy        2         2         2       2            2           kubernetes.io/os=linux   18m

NAMESPACE     NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns          2/2     2            2           18m
kube-system   deployment.apps/metrics-server   1/1     1            1           3m40s

NAMESPACE     NAME                                       DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-77d6fd4654         2         2         2       18m
kube-system   replicaset.apps/metrics-server-54bf7cdd6   1         1         1       3m40s
```

You can make use of the below commands to check the metrics:

```bash
$ kubectl top node          # gives metrics of all the nodes in the K8s cluster

NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   195m         1%     875Mi           1%        
node01         32m          0%     151Mi           0% 

$ kubectl top pod    # gives metrics of all the pods in the K8s cluster

NAME       CPU(cores)   MEMORY(bytes)   
elephant   13m          30Mi            
lion       1m           16Mi            
rabbit     103m         250Mi  
```

------

### Managing Application Logs

Various logging framework

#### How to get the logs from a container within a pod?

Let's take an example:

* When there is just one container in the pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator-application
      image: event-simulator-image
```
```bash
$ kubectl logs -f <pod-name>
```
* When there is multiple containers in the pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator-application
      image: event-simulator-image
    - name: image-processor-application
      image: image-processor-image
```

```bash
$ kubectl logs -f <pod-name> <container-name>
```

Please note, `-f` option will stream/tail the logs from that relevant containers.

#### K8s Reference Documentation:

- https://kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/
- https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/

----------

## :: Application Lifecycle Management

----------

### Rolling Updates and Rollbacks

By default, it is a rolling update strategy:

- Recreate - Every old pods are down and bring up a new set of pods, in this there is a downtime.
- RollingUpdate - In this strategy, it will destroy one pod or several pods at a time, but NOT all the pods, making sure that the other pods are accessible and online to manage the ongoing traffic.


```plaintext
Configuring applications comprises understanding the following concepts:

- Configuring Commands and Arguments on applications

- Configuring Environment Variables

- Configuring Secrets

```

### Application Commands Arguments and Entry points in Containers + Docker + Pods

Please note, the `Command` i.e. `CMD` option is used to inform the container to perform a specific task. For example:

```docker.yaml
CMD ["sleep 5"]
```

So, what happens is, once the container starts running, it will execute this CMD that we added.

But, if you have an `ENTRYPOINT` option, then it is the first executable function so, anything else comes after this command is executed after its executed. For example:

```docker.yaml
ENTRYPOINT ["sleep"]

CMD ["5"]
```

Let's take this command - `docker run Ubuntu <ENTRYPOINT> <CMD>` translates to `docker run Ubuntu sleep 10` so, basically, if you add something in `CMD`, then it automatically gets overrided but, however, the `ENTRYPOINT` does not and it runs as a first executable when the container is restarted or started every time.

You can also override the `ENTRYPOINT` option by just adding the option in the command - `docker run --entrypoint /bin/sh Ubuntu 5`

### Commands ad Arguments in a K8s Pod

Same as you see above in Docker, this works the same in Pods but, with a slight modification:

Comparing with this command: `docker run Ubuntu sleep 10`
```pod-definition.yaml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu
spec:
  containers:
    - image: Ubuntu
      name: ubuntu
      command: ["sleep"]       
      args: ["10"]
```

>[!Caution]
> If you see, the `CMD` in the docker file, is `args` in the K8s pod definition file, whereas, the `ENTRYPOINT` in the docker file is the `command` in the K8s pod definition file, so be careful in understanding this change.

As an alternative method, I tried this and it worked as expected:

```pod-definition.yaml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu
spec:
  containers:
    - image: Ubuntu
      name: ubuntu
      command": ["sleep","10"]       
```

Also, this works too:

```pod-definition.yaml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu
spec:
  containers:
    - image: Ubuntu
      name: ubuntu
      command: ["sleep"]
      args: ["10"]  
```

or, even this is accepted:


```pod-definition.yaml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu
spec:
  containers:
    - image: Ubuntu
      name: ubuntu
      command: 
      - "sleep"   <------------> It should be a string so quotes are mandatory "sleep"
      - "10"
```
-------

### Configure Environment Variables

Taking the same docker for instance:

```bash
$ docker run -e APP_COLOR=pink simple-webapp-color
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
      - containerPort: 8080
      env:
      - name: APP_COLOR
        value: pink
```

Environment value can be added in three ways:

* Plain Key Value
- adding it as `value` itself directly in the pod-definition file.

```yaml
env:
  - name: APP_COLOR
    value: pink
```
* ConfigMaps
- adding it as a `valueFrom` property and referencing it from the `configMap`.

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
```

* Secrets:
- adding it  as a `valueFrom` property and referencing it from the `secrets`.

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```

#### How to work with Configuration Files or ConfigMaps

It is form of key-value pairs and passed to the pods to make use of it.

Two phases:

Create configMap
Inject the configMap into the pod

Two ways to deploy the configMaps:

:: Imperative way 

```bash
$ kubectl create configmap <config-name> --from-literal=<key>=<value>
```

```bash
$ kubectl create configmap app-config --from-literal=APP_COLOR=blue
```

`--from-literal` is used to specify the `key-value` pair and you can use `--from-literal` multiple times, if you have multiple `key-value` pair.

```bash
$ kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod
```
However, adding more `--from-literal` is not the right approach, instead we can pass the `filename` where the `key-value` pairs are stored.

```bash
$ kubectl create configmap app-config --from-file=app_config.properties
```

:: Declarative way:

For this, we create using the `Definition` file:

```yaml
# config-map.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```yaml
kubectl create -f config-map-definition.yaml
```

Example of the ConfigMaps:

```yaml
# app-config.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```
```yaml
# mysql-config.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  port: 3306
  max_allowed_packet: 128M
```
```yaml
# redis-config.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
data:
  port: 6379
  rdb-compression: yes
```

To view the configMaps:

```bash
$ kubectl get configmaps

# output response
NAME                DATA   AGE
db-config           3      7m16s
kube-root-ca.crt    1      25m
webapp-color        2      115s
webapp-config-map   2      8s
```
```bash
$ kubectl describe configmaps db-config

# output response
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_NAME:
----
SQL01

DB_PORT:
----
3306

DB_HOST:
----
SQL01.example.com


BinaryData
====

Events:  <none>
```

Now that we created a `configmaps`, let's inject the configMap to a corresponding pod:

* Default method:

```yaml
# app-config.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
      - containerPort: 8080
      envFrom:
      - configMapRef:
          name: app-config
```

Also, there are different ways to inject environmental variables to a POD:

* Using `Single env`, where we are only passing the `value` through the configMap file but, the `key` is provided in the `pod-definition.yaml` itself:

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
      - containerPort: 8080
      env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:      <---------> Please note, this is configMapKeyRef and NOT just configMapRef
            name: app-config
            key: APP_COLOR
```

* You can make use of the `volume` to add the `Enviromental Variables` to a pod:

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
      - containerPort: 8080
      volumes:
      - name: app-config-volume
        configMap:
          name: app-config
```

#### The Limitations of the ConfigMap:

- Even though, we have configMaps to store the environmental variables like, `username`, `userId`, some important variables, but it is NOT the best place to store the `passwords`. This is where, we need something called as `Secrets`. 
----

### K8s Secrets

Secrets are the place/object where you store the (sensitive information) `passwords` or `keys` and they're similar as `configMaps`. The primary reason why we use the `Secrets` is because the variables are stored in a form of encoded form.

As `configMaps`, there are two steps involved in Secrets:

**Step 1: Create Secret**

  * Imperative way - without using a `secret-definition-file.yaml`

    ```bash 
    # syntax
    $ kubectl create secret generic <secret-name> --from-literal=<key>=<value>
    ```

    ```bash
    $ kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=paswrd
    ```
    ```bash
    # syntax
    $ kubectl create secret generic <secret-name> --from-file=<path-to-file>
    ```

    ```bash
    $ kubectl create secret generic app-secret --from-file=app_secret.properties
    ```

  * Declarative way - with using a `secret-definition-file.yaml`

    ```yaml
    # secret-data.yaml

    apiVersion: v1
    kind: Secret
    metadata:
      name: app-secret
    data:
      DB_Host: mysql
      DB_User: root
      DB_Password: paswrd
    ```
    Convert the `data` in an encoded form i.e. `base64`

    So, you would need to manually `encode` and `decode` the `secret's data value`

    You can make use of the below command to `encode`:

    ```bash
    echo -n "mysql" | base64
    bXlzcWw=
    echo -n "root" | base64
    cm9vdA==
    echo -n "paswrd" | base64
    cGFzd3Jk
    ```

    ```yaml
    # secret-data.yaml

    apiVersion: v1
    kind: Secret
    metadata:
      name: app-secret
    data:
      DB_Host: bXlzcWw=
      DB_User: cm9vdA==
      DB_Password: cGFzd3Jk
    ```
    
    You can make use of the below command to `decode`:   

    ```bash
    echo -n "bXlzcWw=" | base64 --decode
    mysql
    echo -n "cm9vdA==" | base64 --decode
    root
    echo -n "cGFzd3Jk" | base64 --decode
    paswrd
    ```

    ```bash
    kubectl create -f secret-data.yaml
    ```
    To view the `Secrets`, run the below `GET` command:

    ```bash
    $ kubectl get secrets

    # Example:
    $ kubectl get secrets
    
    # Output:
    NAME              TYPE                                  DATA   AGE
    dashboard-token   kubernetes.io/service-account-token   3      2m1s
    ```

    ```bash
    $ kubectl describe secrets <secrets-name>

    # Example:
    $ kubectl describe secrets dashboard-token

    # Output:
    Name:         dashboard-token
    Namespace:    default
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name: dashboard-sa
                  kubernetes.io/service-account.uid: 07edd938-81d1-4cf8-a988-c5c506464493

    Type:  kubernetes.io/service-account-token

    Data
    ====
    ca.crt:     566 bytes
    namespace:  7 bytes
    token:      ey.......hLg
        
    ```

**Step 2: Inject the Secret to the pod**

```yaml
# secret-data.yaml

apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: paswrd
```

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - secretRef:
            name: app-secret
```
```bash
$ kubectl create -f pod-definition.yaml
```

Different ways to inject secrets to Pod:

- Default method:

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - secretRef:
            name: app-secret
```

- Calling just a `single Enviromental Variable`

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: DB_Password
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_Password
```

- Injecting the secrets via the `Volumes`

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
        - containerPort: 8080
      volumes:
      - name: app-secret-volume
        secret:
          secretName: app-secret
```

Here, when using the `Secrets` as `Volumes`, each `key-value` is created as a `key` the file and the `value` as the data within the file.

Once, you `ssh` within the `container`:

```bash
$ ls /opt/app-secret-volumes

# output
DB_Host   DB_Password   DB_User
```

```bash
$ cat /opt/app-secret-volumes/DB_Password

# output
paswrd
```

>[!Important]
> * Please note, the `Secrets` are NOT `encrypted`, just `encoded` so, keep it in mind because anyone can `decode` the secrets.
> * Do not push it along with the SCM(Source Code Management) in Github
> * Also, `Secrets` are NOT `encrypted` in ETCD so consider enabling `Encrypting data in REST`.
> * Anyone who creates pods and deployments in the same `namespace`, can access the `Secrets` as well. So, please make sure you have a RBAC(Role-based access control) to restrict the `secrets` from using the specific pods and NOT all.
> * As an advanced method, you can make use of storing the `Secrets` in a `third-party secrets store providers` like AWS Provider, Azure Provider, GCP Provider.

---- 

>[!Important]
> **ADDITIONAL RESOURCES:**
> Dive deep into the world of Kubernetes security with our comprehensive guide to Secret Store CSI Driver.
> https://www.youtube.com/watch?v=MTnQW9MxnRI
> * There are many secrets provider or tools to handle the `secrets` in a `k8s cluster`, tools like - `External Secrets Operator[ESO]`, `Sealed Secrets`, and `Secret Store CSI Driver`.


#### Let's check what, why, and How to use the  `Secret Store CSI Driver` and to manage within the `K8s cluster`

##### Why was this created? What problems does this tool address?

:white_check_mark: Basically, the `Secrets Store CSI Driver` synchronizes secrets from external APIs(like AWS Secrets Manager) and mounts them into pods i.e. containers as `volumes`.

:white_check_mark: The primary reason why, we do this is to manage all secrets in a central place like `Hashicorp Vault` or `AWS Secrets Manager` and it is pulled `dynamically` and not managed `natively` or `locally` within the `k8s pods`.

:white_check_mark: These Secrets are fetched only during the `runtime` i.e. only when the application is executed or a specific function is executed so that it needs the `secrets` to complete the execution successfully, so you don't need to check secrets in `Git` because it is not stored in the `pod` or `secrets`.

##### How does this `Secret Store CSI Driver` work?


We have our `k8s cluster` and we'll install the `Secret Store CSI Driver` in it.

> Helm is a package Manager and using that, we are installing the `secret store csi driver`.

This is where, you install and follow the installation of `Secrets Store CSI Driver`: https://secrets-store-csi-driver.sigs.k8s.io/getting-started/installation.

---- 

### Demo on `Encrypting Secret Data at Rest`

You can verify whether the secrets stored in the `ETCD Data Store` is encrypted or NOT encrypted using this [official K8s documentation](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

```bash
# This command from the above documentation, gives us the output whether the secret is stored exposed as PlainText or Encrypted.

$ ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/secret1 | hexdump -C
```

So, once you notice that `secret` is stored in `plainText`, then we need to make it `Encrypted`.

> [!Important]
> Default: The `encryption` will only apply to the `secrets` or `resources` that are created after the `encrytion at rest` is enabled.
> However, to make the previously created `secrets` to abide with the `encryption at rest`, we need to manually `update` the `secrets` so that it gets encrypted.

To create a random 32-byte base64 private key to perform the `encryption`, you can use the `built-in` command:

```bash
# This command creates a 32-byte key
$ head -c 32 /dev/urandom | base64
```

>[!Important]
> Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered not very safe.
> The concept of safety of the Secrets is a bit confusing in Kubernetes. The Kubernetes documentation page and a lot of blogs out there refer to secrets as a “safer option” to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion, it’s not the secret itself that is safe, it is the practices around it.
> Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:
>
> * Not checking in secret object definition files to source code repositories.
> * Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD.
>
> Also, the way Kubernetes handles secrets. Such as:
>
> * A secret is only sent to a node if a pod on that node requires it.
> * Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
> * Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
>
>Read about the [protections](https://kubernetes.io/docs/concepts/configuration/secret/#protections) and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks).
>
> Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, and [HashiCorp Vault](https://www.vaultproject.io/).
----

### Multi-Container Pods

The reason why we use `Multi-Container` pods is because, we want the `containers` to scale up and scale down together. For example, let's make use of a scenario, where we have a `web-server` and it wants to send the Apache logs to an external monitoring application like `Elasticsearch`, in this situation we can make use of a `Log Agent` container along with the `web-server` container so that it can send logs whenever there is a new instance is up and online.

This removes the explicit intervention of adding a new pod and then, attaching it with the `web-server` pod/service(as it will be in different networks) and then, storing it in a `separate volume` and so on. 

Whenever, you have multiple containers within the same pod, it makes the networking simple between them by exchanging information via the `localhost`.

```yaml
# simple-webapp-with-logs-definition-file.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-with-logs
spec:
  containers:
    - name: simple-webapp-with-logs
      image: simple-webapp-with-logs
      ports:
        - containerPort: 8080
    - name: log-agent
      image: log-agent
```
Here the `log-agent` container can communicate with the `webapp` container using the `localhost:8080`  directly.

[Important official K8s documentation - communicate-containers-same-pod-shared-volume](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)

```yaml
# From official k8s documentation:
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:

  restartPolicy: Never

  volumes:
  - name: shared-data
    emptyDir: {}

  containers:

  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
```
There are 3 common patterns when it comes to designing multi-container PODs.

The first, and what we just saw with the logging service example, is known as a `sidecar` pattern. 
The others are the `adapter` and the `ambassador` pattern.

However, these fall under the `CKAD curriculum` and are not required for the `CKA exam`. So, we will discuss these in more detail in the CKAD course.

![logging_service](multi-container-pod-logs-push-architecture.png)

----

### InitContainers

Let's talk about `InitContainers`, it is very important and it plays a vital role when deploying large-scale applications because it takes care of all the `setup` operations like `pulling the images`, `configurations`, and most importantly, `generating and storing TLS/SSL certificates`. Let's take an example of our `Elasticsearch` itself, when the nodes are starting up, it expects the TLS/SSL certs to be supplied so that it can communicate with eachother securely. So, the TLS/SSL certificates should be generated before-hand so that it can be passed while it's booting up. In this scenario, the `InitContainers` are much helpful where, it takes care of all the certs generation.

Only if the `InitContainer` completes it's task successfully, the `main container` or the `other containers` are started so basically, it is executed `Sequentially` and it should be `successful` to go for the next.

----

In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle.

For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times.

The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fail, the POD restarts.

But at times you may want to run a process that runs to completion in a container. For example, a process that pulls a code or binary from a repository that will be used by the main web application.

That is a task that will be run only one time when the pod is first created. Or a process that waits for an external service or database to be up before the actual application starts.

That's where `initContainers` comes in. An `initContainer` is configured in a pod-like all other containers, except that it is specified inside a initContainers section, like this:


```yaml
# initcontainer-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone  ;']

```

#### An example fetched from ChatGPT:

Your app needs secrets or certificates that are dynamically created or retrieved before it starts.

```yaml
# example-initcontainer-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  containers:
    - name: app
      image: mysecureapp:latest
      volumeMounts:
        - name: certs
          mountPath: /etc/certs
  initContainers:
    - name: generate-cert
      image: cert-gen:latest
      command:
        - sh
        - -c
        - |
          openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
            -keyout /certs/tls.key -out /certs/tls.crt \
            -subj "/CN=myapp/O=myorg";
      volumeMounts:
        - name: certs
          mountPath: /certs
  volumes:
    - name: certs
      emptyDir: {}
```

* **How It Works:**
  - The Init Container generates TLS certificates using openssl.
  - The certificates are stored in a shared volume.
  - The main app uses the certificates to enable secure communication.

> [!TIP]
> You can just use the `kubectl describe pod` instead of specifying a specific `<pod-name>`, this returns the complete description of all the pods in the `namespace`.

> [!NOTE]
> Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps ensure that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.
> Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes. However, these are not required for the CKA exam and as such, they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.

----------

## :: Cluster Maintenance

----------

### Operating System Upgrade

```bash
$ kubectl drain node-1
```

This command will drain i.e. remove all the pods from the `node-1` and then, `node-1` is left with no workloads i.e. no pods. So, the pods are moved to the other nodes. Technically, in the backend, the pods are NOT moved, it is gracefully Terminated within the `node-1` and then, `recreated` in `node-2` or other nodes.

>[!Important]
> Once you drain the pods from the `node-1`, the node is marked as `UnSchedulable` and [`Cordon`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_cordon/).
> Cordon means: In Kubernetes, the `kubectl cordon` command marks a node as `unschedulable`, preventing new pods from being scheduled on it.
> Once the nodes are removed and added again, you would need to `uncordon` the `node-1` to schedule the new pods on the node.
> You need to use `kubectl uncordon node-1` to remove the `cordon` marked in the `node-1`.


#### Example Scenario:

Vacating all the pods from the `node01`:

```bash
# Before vacating the pods from the node - node01

$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
blue-56dd475db5-6dhgt   1/1     Running   0          4m1s   172.17.1.2   node01         <none>           <none>
blue-56dd475db5-6xbjl   1/1     Running   0          4m1s   172.17.0.4   controlplane   <none>           <none>
blue-56dd475db5-c6zt8   1/1     Running   0          4m1s   172.17.1.3   node01         <none>           <none>
```

```bash
# Removing all the pods from the node - node01

$ kubectl drain node01 --force --ignore-daemonsets=true
node/node01 already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-cr8sj, kube-system/kube-proxy-gmnmv
evicting pod default/blue-56dd475db5-c6zt8
evicting pod default/blue-56dd475db5-6dhgt
pod/blue-56dd475db5-6dhgt evicted
pod/blue-56dd475db5-c6zt8 evicted
node/node01 drained

# Post the cordoned:

$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
blue-56dd475db5-6xbjl   1/1     Running   0          6m1s   172.17.0.4   controlplane   <none>           <none>
blue-56dd475db5-9lq2m   1/1     Running   0          105s   172.17.0.6   controlplane   <none>           <none>
blue-56dd475db5-jpmlk   1/1     Running   0          105s   172.17.0.5   controlplane   <none>           <none>

# Now, checking the number of nodes:

$ kubectl get nodes
NAME           STATUS                     ROLES           AGE     VERSION
controlplane   Ready                      control-plane   10m     v1.31.0
node01         Ready,SchedulingDisabled   <none>          9m35s   v1.31.0

# Uncordoning the node to make it available for scheduling the pods

kubectl uncordon node01
node/node01 uncordoned


# Now, the node01 is `READY`:

kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
controlplane   Ready    control-plane   13m   v1.31.0   192.168.219.165   <none>        Ubuntu 22.04.4 LTS   5.15.0-1072-gcp   containerd://1.6.26
node01         Ready    <none>          13m   v1.31.0   192.168.212.139   <none>        Ubuntu 22.04.4 LTS   5.15.0-1070-gcp   containerd://1.6.26

```

```bash
$ kubectl get node node01 -o yaml >> node01_post_adding_cordon.yaml
cat node01_post_adding_cordon | grep -e "unschedulable"
...
...
    key: node.kubernetes.io/unschedulable
  unschedulable: true
    key: node.kubernetes.io/unschedulable
  unschedulable: true
...
...
```

Now, it is made schedulable. But, please note, only when the pods are created those pods are scheduled accordingly. So, basically, when you `drain` the nodes, the `taints` are added making the node `unschedulable` to `true` and, then it is removed when we `uncordon` it.


----

### Kubernetes Releases and Functions:

```bash
$ kubectl get nodes
```
#### Release Versions:
v1.11.3
Major.Minor.Patch
While Minors are released every few months.
Features and Functionalities

Patches are released every few weeks or more frequently.
Bug fixes.

`v1.10.0-alpha` - alpha releases are mostly buggy and features are disabled.
`v1.10.0-beta` - beta releases are where the code is well tested and features are enabled, it's make way to the `major.minor.patch`.

>[!Important]
> Control-Plane Components:
> kubeapi-server - v1.13.4
> kube-scheduler - v1.13.4
> controller-manager - v1.13.4
> kubelet - v1.13.4
> kube-proxy - v1.13.4
> kubectl - v1.13.4
> But,
> ETCD Cluster - v3.2.18
> CoreDNS - v1.1.3

Note, the `ETCD Cluster` and `CoreDNS` are different projects so that the versions of those components are different.

#### Standard software release function and References:

- https://kubernetes.io/docs/concepts/overview/kubernetes-api/

Here is a link to Kubernetes documentation if you want to learn more about this topic (You don’t need it for the exam, though):

- https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md

- https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md

----

### Cluster Upgrade Process

> [!Warning]
> * Only the latest 3 Major.Minor version are supported in K8s cluster
> * For example, if current latest/stable version release is `v1.13`, and you're cluster is in `v1.10` it is `unsupported` so, you would need to `upgrade` to `v1.11` to be `supported`. So, the supported versions are `v1.11`, `v1.12`, and `v1.13`.
> * **Highly Recommended:** Please follow the `kubeadm-upgrade` documentation: https://v1-31.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/. Documentations are really important and helpful.

* Steps to upgrade:
  - Prerequisites before upgrading: Please note, K8s maintains separate package repositories for each `Minor` versions(ex: upgrading from 1.30 > 1.31), so you would need to update the `package repository` before upgrading each time. Please refer to this [official documentation](https://v1-31.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/) for more information.
  - Upgrade `kubeadm` first
  - Upgrade `controlplane` node(all components, except the `ETCD datastore` and `CoreDNS`, you can upgrade these two only after upgrading all the controlplane nodes and its core components)
  - Upgrade `kubelet` and `kubectl` of the controlplane node
  - Upgrade `worker nodes` & upgrade `kubelet` and `kubectl` of the controlplane node.


> Master node upgrade, does not affect running worker nodes but just the management functions are affected.

* Strategy 1 - ReCreate
* Strategy 2 - Rolling Update using `drain` and `cordon`    --- # Recommended
* Strategy 3 - Add new upgraded nodes to the cluster directly and decommission the old ones but, you can move the workloads to the new nodes before decommisioning.


##### Commands Used:

```bash
$ vi /etc/apt/sources.list.d/kubernetes.list

deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /

Please note, you would need to modify the version v1.30 to the Minor version you would like to upgrade to i.e. v1.31 so that the package repositories are pulled from that version
```

```bash
# You can check the available versions of the kudeadm or kubelet version from the package repository using this command:

$ sudo apt-cache madison kubeadm

   kubeadm | 1.31.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages

$ apt-cache madison kubelet

   kubelet | 1.31.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubelet | 1.31.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubelet | 1.31.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubelet | 1.31.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubelet | 1.31.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
```

## Very Important Commands Before Upgrading:

```bash
$ kubeadm version

# Sample output response:
kubeadm version: &version.Info{Major:"1", Minor:"31", GitVersion:"v1.31.0", GitCommit:"9edcffcde5595e8a5b1a35f88c421764e575afce", GitTreeState:"clean", BuildDate:"2024-08-13T07:35:57Z", GoVersion:"go1.22.5", Compiler:"gc", Platform:"linux/amd64"}
```

```bash
$ kubeadm upgrade plan

# Sample output response:

kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.31.0
[upgrade/versions] kubeadm version: v1.31.0
I0105 01:27:56.380091   40368 version.go:261] remote version is much newer: v1.32.0; falling back to: stable-1.31
[upgrade/versions] Target version: v1.31.4
[upgrade/versions] Latest version in the v1.31 series: v1.31.4

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.31.0   v1.31.4        ########## Both the nodes upgraded successfully to v1.31.0 from v1.30.0
kubelet     node01         v1.31.0   v1.31.4        ########## Both the nodes upgraded successfully to v1.31.0 from v1.30.0

Upgrade to the latest version in the v1.31 series:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.31.0    v1.31.4
kube-controller-manager   controlplane   v1.31.0    v1.31.4
kube-scheduler            controlplane   v1.31.0    v1.31.4
kube-proxy                               1.31.0     v1.31.4
CoreDNS                                  v1.11.1    v1.11.1
etcd                      controlplane   3.5.15-0   3.5.15-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.31.4

Note: Before you can perform this upgrade, you have to update kubeadm to v1.31.4.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```

```bash
# replace x in 1.31.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.0-1.1' && \
sudo apt-mark hold kubeadm

Canceled hold on kubeadm.
Hit:2 https://download.docker.com/linux/ubuntu jammy InRelease                                                                                                
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.31/deb  InRelease                                                       
Hit:3 http://archive.ubuntu.com/ubuntu jammy InRelease                                                
Get:4 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Get:5 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Hit:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease  
Fetched 257 kB in 1s (295 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 25 not upgraded.
Need to get 11.4 MB of archives.
After this operation, 8040 kB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.31/deb  kubeadm 1.31.0-1.1 [11.4 MB]
Fetched 11.4 MB in 0s (46.7 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 13733 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.31.0-1.1_amd64.deb ...
Unpacking kubeadm (1.31.0-1.1) over (1.30.0-1.1) ...
Setting up kubeadm (1.31.0-1.1) ...
kubeadm set on hold.
```

and,

```bash
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.0-1.1' kubectl='1.31.0-1.1' && \
sudo apt-mark hold kubelet kubectl

Canceled hold on kubelet.
Canceled hold on kubectl.
Hit:2 https://download.docker.com/linux/ubuntu jammy InRelease                                   
Hit:3 http://archive.ubuntu.com/ubuntu jammy InRelease                                                 
Hit:4 http://security.ubuntu.com/ubuntu jammy-security InRelease                                       
Hit:5 http://archive.ubuntu.com/ubuntu jammy-updates InRelease                                         
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.31/deb  InRelease
Hit:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 24 not upgraded.
Need to get 26.4 MB of archives.
After this operation, 18.3 MB disk space will be freed.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.31/deb  kubectl 1.31.0-1.1 [11.2 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.31/deb  kubelet 1.31.0-1.1 [15.2 MB]
Fetched 26.4 MB in 0s (69.4 MB/s) 
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 13733 files and directories currently installed.)
Preparing to unpack .../kubectl_1.31.0-1.1_amd64.deb ...
Unpacking kubectl (1.31.0-1.1) over (1.30.0-1.1) ...
Preparing to unpack .../kubelet_1.31.0-1.1_amd64.deb ...
Unpacking kubelet (1.31.0-1.1) over (1.30.0-1.1) ...
dpkg: warning: unable to delete old directory '/etc/sysconfig': Directory not empty
Setting up kubectl (1.31.0-1.1) ...
Setting up kubelet (1.31.0-1.1) ...
kubelet set on hold.
kubectl set on hold.
```


## Commands to execute the upgrade:

```bash
$ kubeadm upgrade apply v1.12.0
$ apt-get upgrade -y kubeadm=1.12.0-00    
$ apt-get upgrade -y kubelet=1.12.0-00
$ kubeadm upgrade node config --kubelet-version v1.12.0
$ systemctl restart kubelet

```

-----

### Backup and Restore

What are the resources will you backup? What are the Backup Candidates?

The backup candidates are the resource configurations and the ETCD Cluster.

> For Resource Configurations, if you use the `Declarative` way to implement the objects/resources, then the files are stored as a definition file accordingly but, in `Imperative` implementation, it does not store the same.
> But, for ETCD Cluster, you can take the `snapshots` using the `snapshot save` option so that it can be taken backup accordingly.
> Also, `Persistent Volumes`.

**WORKING WITH ETCDCTL**

`etcdctl` is a command line client for `etcd`.

In all our `Kubernetes` Hands-on labs, the `ETCD key-value` database is deployed as a static pod on the master. The version used is `v3`.

To make use of `etcdctl` for tasks such as `back up and restore`, make sure that you set the `ETCDCTL_API` to `3`.

You can do this by exporting the variable `ETCDCTL_API` prior to using the `etcdctl` client. This can be done as follows:

```bash
export ETCDCTL_API=3
```

On the Master Node:

![master_node_etcd](master_node_etcd.png)

To see all the options for a specific sub-command, make use of the `-h` or `–help` flag.

For example, if you want to take a `snapshot` of `etcd`, use:

`etcdctl snapshot save -h` and keep a note of the mandatory global options.

Since our `ETCD database` is `TLS-Enabled`, the following options are mandatory:

```
–cacert               verify certificates of TLS-enabled secure servers using this CA bundle

–cert                  identify secure client using this TLS certificate file

–endpoints=[127.0.0.1:2379] This is the default as ETCD is running on master node and exposed on localhost 2379.

–key                 identify secure client using this TLS key file

```
#### Taking Snapshot Using ETCDCTL Options:

Refer to this official documentation: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#snapshot-using-etcdctl-options

```bash

export ETCDCTL_API=3

# Command used to backup i.e. `snapshot ETCD`

etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert="/etc/kubernetes/pki/etcd/ca.crt" --cert="/etc/kubernetes/pki/etcd/server.crt" --key="/etc/kubernetes/pki/etcd/server.key" \
  snapshot save /opt/snapshot-pre-boot.db
```

While restoring the snapshot:

```bash
$ cat etcd-backup-and-restore.md 
1. Get etcdctl utility if it's not already present.
go get github.com/coreos/etcd/etcdctl

2. Backup
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
          snapshot save /opt/snapshot-pre-boot.db

          -----------------------------

          Disaster Happens

          -----------------------------
3. Restore ETCD Snapshot to a new folder
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --name=master \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     --data-dir /var/lib/etcd-from-backup \
     --initial-cluster=master=https://127.0.0.1:2380 \
     --initial-cluster-token etcd-cluster-1 \
     --initial-advertise-peer-urls=https://127.0.0.1:2380 \
     snapshot restore /opt/snapshot-pre-boot.db

 4. Modify /etc/kubernetes/manifests/etcd.yaml
 Update --data-dir to use new target location
 --data-dir=/var/lib/etcd-from-backup

 Update new initial-cluster-token to specify new cluster
 --initial-cluster-token=etcd-cluster-1

 Update volumes and volume mounts to point to new path
      volumeMounts:
          - mountPath: /var/lib/etcd-from-backup
            name: etcd-data
          - mountPath: /etc/kubernetes/pki/etcd
            name: etcd-certs
   hostNetwork: true
   priorityClassName: system-cluster-critical
   volumes:
   - hostPath:
       path: /var/lib/etcd-from-backup
       type: DirectoryOrCreate
     name: etcd-data
   - hostPath:
       path: /etc/kubernetes/pki/etcd
       type: DirectoryOrCreate
     name: etcd-certs

```
---------------------
Steps to Restore:



##### Restore when you are in a controlplane:

First Restore the snapshot:
root@controlplane:~# ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
snapshot restore /opt/snapshot-pre-boot.db


2022-03-25 09:19:27.175043 I | mvcc: restore compact to 2552
2022-03-25 09:19:27.266709 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
root@controlplane:~# 


Note: In this case, we are restoring the snapshot to a different directory but in the same server where we took the backup (the controlplane node) As a result, the only required option for the restore command is the --data-dir.

Next, update the `/etc/kubernetes/manifests/etcd.yaml`:

We have now restored the etcd snapshot to a new path on the controlplane - /var/lib/etcd-from-backup, so, the only change to be made in the YAML file, is to change the hostPath for the volume called etcd-data from old directory (/var/lib/etcd) to the new directory (/var/lib/etcd-from-backup).

```yaml
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```

With this change, `/var/lib/etcd` on the container points to `/var/lib/etcd-from-backup` on the controlplane (which is what we want).

When this file is updated, the ETCD pod is automatically re-created as this is a static pod placed under the /etc/kubernetes/manifests directory.

`Note 1`: As the ETCD pod has changed it will automatically restart, and also `kube-controller-manager` and `kube-scheduler`. Wait 1-2 to mins for this pods to restart. You can run the command: `watch "crictl ps | grep etcd"` to see when the ETCD pod is restarted.

`Note 2`: If the etcd pod is not getting Ready 1/1, then restart it by kubectl delete pod -n kube-system etcd-controlplane and wait 1 minute.

`Note 3`: This is the simplest way to make sure that ETCD uses the restored data after the ETCD pod is recreated. You don't have to change anything else.

If you do change `--data-dir` to `/var/lib/etcd-from-backup` in the ETCD YAML file, make sure that the volumeMounts for `etcd-data` is updated as well, with the mountPath pointing to `/var/lib/etcd-from-backup` (THIS COMPLETE STEP IS OPTIONAL AND NEED NOT BE DONE FOR COMPLETING THE RESTORE)

We can run `kubectl logs etcd-controlplane -n kube-system` command to get the details of the etcd logs.
------------------------

> [Important]
> How to find the number of clusters in a node?
```bash
$ kubectl config get-clusters
NAME
cluster1
cluster2

$ kubectl config use-context cluster1
Switched to context "cluster1".

$ kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
cluster1-controlplane   Ready    control-plane   50m   v1.29.0
cluster1-node01         Ready    <none>          49m   v1.29.0

$ kubectl config use-context cluster2
Switched to context "cluster2".

$ kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
cluster2-controlplane   Ready    control-plane   51m   v1.29.0
cluster2-node01         Ready    <none>          51m   v1.29.0

# check whether the `etcd-server` is configured or deployed as pod in the cluster or deployed separately or it is not deployed at all.
control-plane $ ps -aux | grep -e "etcd"
root        3070  0.0  0.1 1562828 293936 ?      Ssl  00:46   2:14 kube-apiserver --advertise-address=192.1.141.14 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem --etcd-servers=https://192.1.141.3:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root       11048  0.0  0.0   6836  2360 pts/0    S+   01:47   0:00 grep -e etcd
```

To do list:

-- Create the high level design of how the etcd -> kube-apiserver is communicating and why it is important to understand this.

- Stores the cluster state, configuration, and metadata.
- Highly available and distributed.
- Interacts with: * API Server (read/write operations).


-----

References K8s documentation:

* https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster 
* https://github.com/etcd-io/website/blob/main/content/en/docs/v3.5/op-guide/recovery.md 
* https://www.youtube.com/watch?v=qRPNuT080Hk

----------

## :: Security

----------

Primitives - Elementary or Fundamentals

### K8s Security Primitives:

In the core-components, we should start with `kube-apiserver`

We should know:

* Who Can Access?

It is defined by the authentication mechanism, there are different ways to authenticate to the `api-server`

- Files - Username and Passwords
- Files - Username and Tokens
- Certificates
- External Authentication Providers - LDAP
- Service Accounts

* What can they do?

Once the user gain Access to control, what can they do? It is defined by the `Authorization` Mechanisms:

- RBAC Authorization
- ABAC Authorization
- Node Authorization
- Webhook Mode

All communications between the various components in the cluster is secured using the `TLS encryptions`:

We can even restrict the pods accessing other pods using the `Network Policies`.

----

### Authentication

All users access are managed by `kube-apiserver`, how does the `kube-apiserver` authenticate?

You can use:

- Static Password File
- Static Token File
- Certificates
- Identity Services, like LDAP & Kerberos

Let's start with a simplest form of authentication:

We can make use of the `Static Password File` and `Static Token File`:

Like for example:

```csv
# static-password-file.csv
user1, password123, u0001
user2, password321, u0002
user3, password092, u0003
```
```csv
# static-token-file.csv
R/buQPCACziovsv80xvfCxEg9S3lbtyoUdg3sC4ZiBE=, password123, u0001
B8eATQX2aZupbqowegAUiIb6+Ive+ERz1DIO5ECL/fs=, password321, u0002
15Wkmy1lmzxbEDSWYe3uFkhP8bUAqZ8x/LuIwhRpnG4=, password092, u0003
```

>[!Important]
> This is NOT a recommended authentication mechanism
> Consider Volume Mount while providing the auth file in a `kubeadm` setup
> Setup Role Based Authorization for the new users

------

### TLS Introduction

Securing a cluster with the TLS certificates is challenging.


Goals:

- What are TLS certificates?
- How does K8s use certificates?
- How to generate them?
- How to configure them?
- How to view them?
- How to troubleshoot issues related to certificates

### TLS Basics

#### TLS Certificates Pre-Requisites

##### Symmetric Encryption:

The Server and the Client using a same key to `encrypt` and `decrypt` the data, so the client has to pass the `key` to the `server` to decrypt so the man-in-the-middle attack is very possible when you pass the key.


##### Asymmetric Encryption:

It uses a `pair` of keys - `Private` key and `Public` key.

Consider `Public key` as a `Public Lock`

```bash
# To generate a private key
$ openssl genrsa -out my-bank.key 1024
my-bank.key

# To generate a public key
$ openssl rsa -in my-bank.key -pubout > mybank.pem
my-bank.key mybank.pem
```

Please note, `ssh key-gen` command format is different and `openssl` format is different.


Important to notice who signed the certificate:

- Self-signed certificate

Public CA

Private CA

- Trusted CA certificate

What does the `Public Key Infrastructure` do?

Let's see step-by-step:

- There are two methods:
 * One for `ssh`
 * One for `https`

##### For `ssh`:

The user can create a `ssh key-gen` where, it will give us a `private key` and `public key` which is known as `id_rsa` and `id_rsa.pub`:

Commands that are used to generate the `ssh certificates`

```bash
$ ssh-keygen -t rsa -b 4096
$ ssh-keygen -t dsa 
$ ssh-keygen -t ecdsa -b 521
$ ssh-keygen -t ed25519
```
So, basically once this `public key` i.e. `id_rsa.pub` is generated, we can `copy-paste` this key in the `server/host` where the applications are hosted so that we can gain access to that `server`.

Once, you type `ssh -i id_rsa user@server`, as you parsed the `private key - id_rsa`, it is easier when the `server` provides you with the `public key` so that you can decrypt and login to the server successfully.

Here is the workflow:

![ssh_key_architecture](ssh_key_architecture.png)


```mermaid
sequenceDiagram
participant Client
participant Server
participant CA

Note over Client,Server: SSH Communication
Client->>Server: Initiates SSH connection
Server-->>Client: Sends public key
Client->>Server: Verifies server's public key (via known_hosts)
Client->>Server: Exchanges symmetric key (via key exchange)
Server-->>Client: Securely establishes SSH session
```

-----

>[!Caution]
> Please note, whenever you see an `http`, you should keep in mind that the `data` that is passed via the `client` and to the `server` is NOT encrypted so basically, it can be exposed to the `man-in-the-middle` attack i.e. if someone `sniffs` the network packets they should know what is the `data` itself. 
> Which is why, we use the `HTTPS` protocol to make sure that the `data` within the Network should be masked/abstracted/hashed/encrypted and it can only decrypted on the `server` side.

##### For `HTTPS`:

To communicate securely with the `server` and the `client`, we make use of the `browsers` here:

`Browsers` play an important role in abstracting the `certificate validation` techniques and process from us.

Here are the steps:

- The `user` enters a domain name `https://www.website.com` in the `browser`, so the moment user enters the domain name, the specific domain goes to the DNS and reaches the specific server that is hosted the domain. It will issue a `server` certificate which is the `public` certificate with the `signed by` values i.e. whether it is `self-signed certificate` or the `Trusted CA certificate`.
- This `public key` is validated by the `Browser` under-the-hood certificate process and then, opens the certificate and gets the `public key`. Now, the browser uses that `CA Trusted public key` and retrieves the `server's public key`.
- Now, by making use of the `server's public key`, it encrypts the `client's private key` i.e. the symmentric key. And, it is sent to the `Server`. Now, the server decrypts the `symmetric key` using it's `server's private key`. Now, this `symmetric key` is used for communication. Basically, the `Browser` generates only one `private key` i.e. the `symmetric key`.

How are the Trusted CA Server Certificates are generated?

- First, the server need to create a `CSR` - Certificate Signing Request using the valid domain name i.e. `Common Name`, and `Subject Alternative Names` (SANs).
- Validation, send it to the `CA` for validation so that it can validate and sign the `CSR`.
- Approved, once the `CA` signed the `CSR`, it issues a `Trusted CA Server Certificate` so that it can be used to present to the `clients`.

Basically, public keys are representated as this:

```
server.crt
client.crt
server.pem
client.pem
```

Private key files are representated as this:

```
server-key.crt
server.key
client-key.crt
client.key
```

Here is the workflow, with a scenario:

I created a website(blogger.com), now I want this website to be secured with `HTTPS`.

- As a prerequisite, I'll use the `openssl genrsa -out private.key 2048` command to create a `server's private key`. Now, you can create a `public key` using the `private key` and sign the `CSR`. 

- First, I'll create a `CSR` with all the `domain name` - blogger.com and the `subject alternative name (SANs)` - blog.blogger.com, xyz.blogger.com, *.blogger.com and send it to the `Trusted CAs` like `Symantec` or `DigiCert`.
- They'll validate the certificate and send me the `signed certificate` so the server i.e. the website can send it to the clients to validate and run a secure connection.
- By fetching the `certificate` from the `server`, it knows the `CA` who signed the certificate and it uses the `CA's public key` and reaches out to the `server` for the `server's public key`. Once, the `server offers the public key`. The client `encrypts` its `symmetric key` i.e. its `client's private key` using the `server's public key` and send it to the `server`. Now, the `server` decrypts the encrypted symmetric key using it's `private key`. So, that, the data between the `client` and the `server` can be hashed using this `symmetric key`.

![https_secure_communication](https_secure_communication.png)

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant CA

    %% HTTPS Communication
    Note over Client,CA: HTTPS Communication
    Client->>Server: Initiates HTTPS connection
    Server-->>Client: Sends SSL/TLS certificate
    Client->>CA: Verifies certificate authenticity
    CA-->>Client: Confirms validity of certificate
    Client->>Server: Exchanges symmetric key (via TLS handshake)
    Server-->>Client: Securely establishes HTTPS session
```
-----

### TLS in kubernetes

Let's see how to secure the K8s cluster using the TLS/SSL certificates:

##### Three certificates:

- Root Certificates
- Server Certificates
- Client Certificates

Basically, public keys are representated as this:

```
server.crt
client.crt
server.pem
client.pem
```

Private key files are representated as this:

```
server-key.crt
server.key
client-key.crt
client.key
```

##### K8s Server Components and its certificates/keys:

- Kube API Server - apiserver.crt, apiserver.key
- etcd Server - etcdserver.crt, etcdserver.key
- kubelet Server - kubelet.crt, kubelet.key

##### Client Components and its certificates/keys:

- Admin - admin.crt, admin.key
- kube-scheduler - scheduler.crt, scheduler.key
- kube-controller-manager - controller-manager.crt, controller-manager.key
- kube-proxy - kube-proxy.crt, kube-proxy.key

![alt text](certificates_and_components.png)

![alt text](server_and_client_certificates.png)

-----

### TLS in K8s - Certificate Creation

##### Let's start with a `CA Certificate`:

```bash
# Generating the CA Private Key
$ openssl genrsa -out ca.key 2048
```

```bash
# Generating CSR Certificates so that the CA can sign
$ openssl req -new -key ca.key -sub "/CN=KUBERNETES-CA" -out ca.csr
```

```bash
# With the CSR, SIGN THE CERTIFICATES and generate a ca.crt
$ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

Basically, the `CA self-signs` the certificate using the `private-key` that is generated above.

Now, this `ca.crt` is the `Root Certificate File` and the `ca.key` is the `private key`.


##### Let's try for the `Admin User`

```bash
# Generating the admin Private Key
$ openssl genrsa -out admin.key 2048
```

```bash
# Generating CSR Certificates so that the CA can sign
$ openssl req -new -key admin.key -sub "/CN=kube-admin" -out admin.csr
OR
# Generating CSR Certificates with the User Group as System:Masters
$ openssl req -new -key admin.key -sub "/CN=kube-admin/O=system:masters" -out admin.csr
```
Note, the `CN` can vary, you should use the relevant value. Also, specifying `system:masters` means that the we are letting only the `Administrators` in the K8s cluster accessing the pods.

[Here is a K8s official documentation on using the `system:masters`](https://kubernetes.io/docs/setup/best-practices/certificates/#:~:text=In%20the%20above,super%20user%20group.)

```bash
# With the CSR, SIGN THE CERTIFICATES and generate a admin.crt
$ openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```
Now, here we are signing the `admin` certificate using the `CA` certificates.


##### Let's create it for `kube-scheduler`

```bash
# Generating the admin Private Key
$ openssl genrsa -out scheduler.key 2048
```

```bash
# Generating CSR Certificates so that the CA can sign
$ openssl req -new -key scheduler.key -sub "/CN=system:kube-scheduler" -out scheduler.csr
```

```bash
# With the CSR, SIGN THE CERTIFICATES and generate a admin.crt
$ openssl x509 -req -in scheduler.csr -CA ca.crt -CAkey ca.key -out scheduler.crt
```

##### Now, for `Controller-Manager`

```bash
# Generating the admin Private Key
$ openssl genrsa -out controller-manager.key 2048
```

```bash
# Generating CSR Certificates so that the CA can sign
$ openssl req -new -key controller-manager.key -sub "/CN=system:kube-controller-manager" -out controller-manager.csr
```

```bash
# With the CSR, SIGN THE CERTIFICATES and generate a admin.crt
$ openssl x509 -req -in controller-manager.csr -CA ca.crt -CAkey ca.key -out controller-manager.crt
```

Please note, we added `system:core_components` name because, those components are a part of the `control plane`.

##### It's now the turn for `Kube-Proxy`

```bash
# Generating the admin Private Key
$ openssl genrsa -out kube-proxy.key 2048
```

```bash
# Generating CSR Certificates so that the CA can sign
$ openssl req -new -key kube-proxy.key -sub "/CN=kube-proxy" -out kube-proxy.csr
```

```bash
# With the CSR, SIGN THE CERTIFICATES and generate a admin.crt
$ openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -out kube-proxy.crt
```

##### Let's see the `Server Side` Certificates

etcdserver certificates

kube-apiserver

```bash
$ openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr --config openssl.cnf
```

```bash
# openssl.cnf
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 192.168.5.11
IP.3 = 192.168.5.12
IP.4 = 192.168.5.30
IP.5 = 127.0.0.1
```

```bash
$ openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -extensions v3_req -extfile openssl.cnf -days 1000
```

Kubelet Server

```
system:node:node01
```
```
group:nodes
```
-----

### View Certificate Details

If the services are configured as `native` services, then use:

```bash
$ journalctl -u etcd.service -l
```

If the services are configured as `pods` while using `kubeadm` then, inspect using the `kubectl` utility.

```bash
$ kubectl logs <pod-name>
```
Sometimes, the core-components like the `kube-apiserver` or `etcd-server` is not working, then the `kubectl` will NOT help.

So, we need to dig through the `docker logs` to identify what happened.

```bash
$ docker ps -a
$ docker logs <container-id>
```

Or, you can use the `crictl logs` to understand what happened, if the `kubectl` is not responding.

Certificate Health Check Spreadsheet I have uploaded the Kubernetes Certificate Health Check Spreadsheet here:

https://github.com/mmumshad/kubernetes-the-hard-way/tree/master/tools

Feel free to send in a pull request if you improve it.

[Kubernetes Certificates Checker Spreadsheet](kubernetes-certs-checker.xlsx)

-----

### Certificates API

How to manage Certificates and What is the Certificates API:

Make use of the `certificate API` i.e. `kubectl certificate <options>` to approve and deny the requests and we do not need to manually sign the `CSRs` using the `CA` as this API does that for us. This way, we know exactly who we gave access to.

Already I have a CSR request here:
```bash
$ cat akshay.csr 
-----BEGIN CERTIFICATE REQUEST-----
MIICVjCCAT4CAQAwETEPMA0GA1UEAwwGYWtzaGF5MIIBIjANBgkqhkiG9w0BAQEF
AAOCAQ8AMIIBCgKCAQEAx2bB1CBAMgBVuQTXXNYapn7sdA2+slGgGP+qGvy6goYY
lqXTe3Z/OCbGnay0TLN3iiAeX8r0Ios41Sq8prLl7vC8uUTRUwMDLiHO8rqQKZfo
yj6VvlnFP41QkADjIomu4J0TfD8Dr0c1JMLb3gctUfmhRWZ6N1Azsczobe3bYojG
6IS4s4ljsUjrUKZB1fgpzJa0lg9B+6wP/gnyBNPWAxUZsR7S7B0bNgSJiOVcLqON
UQIjFDS7jra14BTvLo7T83xxgA0s24rPjCIWc6z9aB4x5lRMEqdE1u0WoDJT7Utr
xl/JQbT2RrMsq1tG1XQ2+tLhB+OtXDAnHNcwPQlKcQIDAQABoAAwDQYJKoZIhvcN
AQELBQADggEBAHqAw5fp74OO98kjrOuH94uEFMHcgij6o3nlLPq9fc7OEvu9TIbC
TOAj9SutgV8pfgxu6YCc4TGCNs+STV9SwbuOdzLeq/DpCFwFJxyFZm+hgfyOF2Vp
F/HfGJ9G61efqbApS7fzPa+YB/2ZyWky9PNAHvK0b3e14s0EkM62LNMSc6k2H7o3
RAIG7lX5KKhtB1JQX8yt0BALyPnd+fCjKl+DMmV9aQsjbSY9XZLQwS+jiImNUKol
MN2RP1zdDn3Sg3GY+BufZRrDAxbIWFra0o4PFPCHSayeAmMIC6H2Fc9tMw49XBIN
xMSsLy8CIS2Sf2Dkr9b1g31842rnIxoi8Ts=
-----END CERTIFICATE REQUEST-----
```
Already I have a private key here:
```bash
$ cat akshay.key 
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDHZsHUIEAyAFW5
BNdc1hqmfux0Db6yUaAY/6oa/LqChhiWpdN7dn84JsadrLRMs3eKIB5fyvQiizjV
KrymsuXu8Ly5RNFTAwMuIc7yupApl+jKPpW+WcU/jVCQAOMiia7gnRN8PwOvRzUk
wtveBy1R+aFFZno3UDOxzOht7dtiiMbohLiziWOxSOtQpkHV+CnMlrSWD0H7rA/+
CfIE09YDFRmxHtLsHRs2BImI5Vwuo41RAiMUNLuOtrXgFO8ujtPzfHGADSzbis+M
IhZzrP1oHjHmVEwSp0TW7RagMlPtS2vGX8lBtPZGsyyrW0bVdDb60uEH461cMCcc
1zA9CUpxAgMBAAECggEABVr2BnSRyM3/v2L5BkAlUdarks/aMOeWj0lRiB1ExJPl
pekg+YpEjy1ibPJqGrvAsIPdkPjOTv2yem4mCCual4zdYrPtk8dLTvI/QL9gNw/C
g6kD5Et8q9DnqqBteTxdpyBs2SvHlW3qCFlX5poGfnLAATFIA8OKieJNFUO7vkUO
OjTHHUMu+WrvNCSXWsrqf2MVZAU4OvLBt5vlhwoHu/kLmkn4D+jQbMSq9cTDRpKD
4EElM2KF/o5b+0674RSvwDYiuz0C72M9f483mQA2MA/JNh8skAtL+agjR8dHmx8C
XmnOf3WmVio973iNirqoY6g3mrn8Q2ktiVmL4gnJAQKBgQDTyWCDpcW8FtpZCkFv
7AY0qcresj6obI3Wg+6DQXzZXtl/tPClf3vLhsL+9uhQBhB2KHo2jAx5wPQITk0E
cqjDILDxVRdhO3az6tWjlaPH6V4i82fQRBd5mWGvrwbRKGLkAZDXzFo/A/hMxCSY
gqBye8+eNOrkt0EMCdH6HarjgQKBgQDxB3g6NhBE9Yw+Fn/yF5NEfHgOPAK92/y6
589hwVA4ZoYYJUtXrOXpjXaPEfJscsgTdv641xnSMzqlU9zxXEcZ+6m/awLcM9MJ
yaoXiHOcP4EF7TarccnqC4NyvDgzD945NCzJKwLJwk87NhByCEINNXT/KhRwFXY7
KK3yiIAe8QKBgCeZecBj2Ksoor8PWdjxit+B9/rX3TtkwMihdpF4Rz3lrElirbRk
N+OAWSmPRTThTWaqft+AJD0HxRlgeqogPknt7OfS/0Ig+jr95+BkdHFGw2FJndml
QbZs5YUR9vZY1NTJLioLfbxWo7DG5AjYM+P0Hof4YQtO8IytsWNQ9MkBAoGAXu01
MZ/nSnF2uWE9JMiaLIfk9J436jSA1c/EQljTUlM2jr3pbW9f4Vrgjbpgwst19wyP
h5bXyYx5lXFw+/H+4sHV+diYeMdEX394KOqJtcjg9MeXF3CqOC4up/pvG9UtDVR6
KwRwhausbsJ9OZv8/5zFJlWZVQdu2B/OWk3eP9ECgYEAthwOv0FO+AunoDA5BC3M
0T6n2jXMDc9QkxzNIswBHwSRkY4UxjdNw+beHKi2Q7pFpaKiZTB/rbHHpqY/HFny
7nvBVb7NnPpZ3nGqJJc8Hpv/yYqzM0lJLrutxL0pLh4mmU6IIUyeskrahrxpsWhl
J64r73gl7Oohxuib0X09OZI=
-----END PRIVATE KEY-----
```
K8s Official documentation on How to Create a `CSR Object`:  https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/#create-a-certificatesigningrequest-object-to-send-to-the-kubernetes-api

```yaml
# csr_definition_file.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  request: $(cat akshay.csr | base64 | tr -d '\n') #### You can convert this to base64 manually.
  signerName: example.com/serving
  usages:
  - digital signature
  - key encipherment
  - server auth
```

Refer to the K8s Documentation on the different types of built-in `signerName`: https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#signers

```bash
$ kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      16s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Pending
csr-sklv6   25m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

$ kubectl certificate approve akshay
certificatesigningrequest.certificates.k8s.io/akshay approved

kubectl get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      4m15s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Approved,Failed
csr-sklv6   29m     kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

kubectl certificate deny agent-smith
certificatesigningrequest.certificates.k8s.io/agent-smith denied

kubectl get csr
NAME          AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
agent-smith   2m51s   kubernetes.io/kube-apiserver-client           agent-x                    <none>              Denied
akshay        8m13s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Approved,Failed
csr-sklv6     33m     kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

kubectl delete csr agent-smith 
certificatesigningrequest.certificates.k8s.io "agent-smith" deleted
```

This is the super important command that can give you all the commands that you are looking for:

```bash
# searching the man pages using the `keyword` search "-k"
$ man -k kubectl

kubectl(1)               - kubectl controls the Kubernetes cluster manager
kubectl-alpha(1), kubectl alpha(1) - Commands for features in alpha
kubectl-annotate(1), kubectl annotate(1) - Update the annotations on a resource
kubectl-api-resources(1), kubectl api-resources(1) - Print the supported API resources on the server
kubectl-api-versions(1), kubectl api-versions(1) - Print the supported API versions on the server, in the form of "group/version"
kubectl-apply(1), kubectl apply(1) - Apply a configuration to a resource by file name or stdin
kubectl-apply-edit-last-applied(1), kubectl apply edit-last-applied(1) - Edit latest last-applied-configuration annotations of a resource/object
kubectl-apply-set-last-applied(1), kubectl apply set-last-applied(1) - Set the last-applied-configuration annotation on a live object to match the contents of a file
kubectl-apply-view-last-applied(1), kubectl apply view-last-applied(1) - View the latest last-applied-configuration annotations of a resource/object
kubectl-attach(1), kubectl attach(1) - Attach to a running container
kubectl-auth(1), kubectl auth(1) - Inspect authorization
kubectl-auth-can-i(1), kubectl auth can-i(1) - Check whether an action is allowed
... 
...
... more
```
------

### KubeConfig

```bash
$ kubectl config view
```

```yaml
# kube config definition file:
$ cat my-kube-config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: development
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: kubernetes-on-aws
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: production
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: test-cluster-1
contexts:
- context:
    cluster: kubernetes-on-aws
    user: aws-user
  name: aws-user@kubernetes-on-aws
- context:
    cluster: test-cluster-1
    user: dev-user
  name: research
- context:
    cluster: development
    user: test-user
  name: test-user@development
- context:
    cluster: production
    user: test-user
  name: test-user@production
current-context: research
kind: Config
preferences: {}
users:
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
```

-----

### API apiGroups

There are multiple API Groups like:

```bash
/metrics
/version
/api
/apis
/healthz
/logs
```

Under these groups, they are categorized into two categories `core` and `named`:

```bash
/api - core group (Now, it's called as Legacy)
/apis - named group
```

Check this [official K8s documentation](https://kubernetes.io/docs/reference/using-api/#:~:text=API%20groups,API%20reference.)

Access the `kube-api` Server:

```bash
$ curl http://localhost:6443 -k
```

```bash
curl http://localhost:6443/apis -k | grep "name"
```

If you would like to access the api like the above, you would need to pass the `admin.crt` and `admin.key` and `ca.crt` file.

And, as an alternate options to run a `kubectl proxy` using the `port 8001`

```bash
curl http://localhost:8001 -k
```

Please note, `kube proxy` and the `kubectl proxy` are NOT the same whereas, the `kubectl proxy` is a HTTP-Server created by the `kubectl` utility to reach the `kube-apiserver`.

----

### Authorization

What is Authorization?

You have got access to the K8s cluster now, but what about the privileges? i.e. Based on the Authentication, you logged in. Now, what are the operations that you can perform on the cluster? This is what, meant as `Authorization`.
You can restrict users to specific resource or actions so that they don't modify or take control of the resource as whole.

There are different types of Authorization:

- Node Based Authorization
- ABAC - Attributes based access control
- RBAC - Role based access control (Recommended)
- Webhook - Using external Policy agents to regulate the privileges

>[!Important]
> Please note, you will need to provide the `authorization mode` in the `kube-apiserver` settings so that you'll instruct what to follow to the `kube-apiserver` when there is a request coming in. In the below snippet, where we notice `--authorization-mode=Node,RBAC` so, `Node` based authorization comes first and then, `RBAC`. Once the `kube-apiserver` gets the request, it will check the `Node` Role and its access controls, if it is restricted, then it will go to `RBAC` based access controls. So, it goes in order.

```yaml
# kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver-controlplane
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.163.5
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC      # Setting the `Authorization Mode`
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    ...
    ...
    image: registry.k8s.io/kube-apiserver:v1.31.0
    imagePullPolicy: IfNotPresent
...
...
```

```mermaid
sequenceDiagram
    participant User
    participant Kube-ApiServer
    participant Node
    participant RBAC
    participant Resource

    %% Authorization
    Note over User,Resource: Authorization
    User->>Kube-ApiServer: Initiates an Authorization request to a resource(s)
    Kube-ApiServer->>Node: Checks whether the User got privileges to access the resource(s)
    Node -->> Kube-ApiServer: No access provided
    Kube-ApiServer->>RBAC: Checks whether the User got access here
    RBAC -->> Kube-ApiServer: No access provided
    RBAC -->> Kube-ApiServer: If yes, then authorized
    Kube-ApiServer -->> User: You're authorized
    User->> Resource: Access provided can perform (kubectl get pods)
```
----

#### RBAC - Role based access control

How do we create a role? again, using an object:

Let's check its requirements:

User: Developers
Operations:
- Can view pods
- Can create pods
- Can delete pods
- Can create configmaps

```yaml
# developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","get","create","update","delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

```bash
$ kubectl apply -f developer-role.yaml
```

Now, the role is created successfully, but we need to associate the `users` i.e. `developers` in this scenario to the `role`.

##### Bonus Tip:

Say, you wanted to give access to a specific pod for a Role instead of all the pods. For example, let's consider there are 5 pods - `blue, green, pink, orange, yellow`, let's try to give access only to `blue` and `green` pod.

```yaml
# developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","get","create","update","delete"]
  resourceNames: ["blue","green"]
```

###### Let's perform Role Binding

```yaml
# devuser-developer-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

An example from the [K8s official documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-example) on `RoleBinding`:

```yaml
# role-binding-example.yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default  # Namespace restriction
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
```bash
$ kubectl apply -f devuser-developer-binding.yaml
```

###### Commands that will be useful post Role and RoleBinding creation

```bash
$ kubectl get roles
$ kubectl get rolebindings
$ kubectl describe role dev-user
$ kubectl describe rolebinding dev-developer-binding
```
As a `user` how can you check access for a specific operation:

```bash
$ kubectl auth can-i create deployments
$ kubectl auth can-i delete nodes
$ kubectl auth can-i create deployments --as dev-user
$ kubectl auth can-i create pods --as dev-user
$ kubectl auth can-i create pods --as dev-user --namespace test
```
```bash
# Usage examples:

$ kubectl get roles -A
NAMESPACE     NAME                                             CREATED AT
blue          developer                                        2025-01-15T00:16:17Z
kube-public   kubeadm:bootstrap-signer-clusterinfo             2025-01-15T00:10:55Z
kube-public   system:controller:bootstrap-signer               2025-01-15T00:10:53Z
kube-system   extension-apiserver-authentication-reader        2025-01-15T00:10:53Z
kube-system   kube-proxy                                       2025-01-15T00:10:56Z
kube-system   kubeadm:kubelet-config                           2025-01-15T00:10:53Z
kube-system   kubeadm:nodes-kubeadm-config                     2025-01-15T00:10:53Z
kube-system   system::leader-locking-kube-controller-manager   2025-01-15T00:10:53Z
kube-system   system::leader-locking-kube-scheduler            2025-01-15T00:10:53Z
kube-system   system:controller:bootstrap-signer               2025-01-15T00:10:53Z
kube-system   system:controller:cloud-provider                 2025-01-15T00:10:53Z
kube-system   system:controller:token-cleaner                  2025-01-15T00:10:53Z

$ kubectl describe role kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]

$ kubectl get role kube-proxy -n kube-system -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2025-01-15T00:10:56Z"
  name: kube-proxy
  namespace: kube-system
  resourceVersion: "232"
  uid: 1b5d32f3-3a02-497c-8a1e-d2b9745e1819
rules:
- apiGroups:
  - ""
  resourceNames:
  - kube-proxy
  resources:
  - configmaps
  verbs:
  - get

$ kubectl get rolebindings -o wide -n kube-system
NAME                                                ROLE                                                  AGE   USERS                                                   GROUPS                                                          SERVICEACCOUNTS
kube-proxy                                          Role/kube-proxy                                       31m                                                           system:bootstrappers:kubeadm:default-node-token                 
kubeadm:kubelet-config                              Role/kubeadm:kubelet-config                           31m                                                           system:nodes, system:bootstrappers:kubeadm:default-node-token   
kubeadm:nodes-kubeadm-config                        Role/kubeadm:nodes-kubeadm-config                     31m                                                           system:bootstrappers:kubeadm:default-node-token, system:nodes   
system::extension-apiserver-authentication-reader   Role/extension-apiserver-authentication-reader        31m   system:kube-controller-manager, system:kube-scheduler                                                                   
system::leader-locking-kube-controller-manager      Role/system::leader-locking-kube-controller-manager   31m   system:kube-controller-manager                                                                                          kube-system/kube-controller-manager, kube-system/leader-election-controller
system::leader-locking-kube-scheduler               Role/system::leader-locking-kube-scheduler            31m   system:kube-scheduler                                                                                                   kube-system/kube-scheduler
system:controller:bootstrap-signer                  Role/system:controller:bootstrap-signer               31m                                                                                                                           kube-system/bootstrap-signer
system:controller:cloud-provider                    Role/system:controller:cloud-provider                 31m                                                                                                                           kube-system/cloud-provider
system:controller:token-cleaner                     Role/system:controller:token-cleaner                  31m                                                                                                                           kube-system/token-cleaner

$ kubectl describe rolebindings kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  kube-proxy
Subjects:
  Kind   Name                                             Namespace
  ----   ----                                             ---------
  Group  system:bootstrappers:kubeadm:default-node-token  

$ kubectl auth can-i get pods --as dev-user
no

$ vi dev-user-role.yaml

$ cat dev-user-role.yaml 
apiVersion: rbac.authorization.k8s.io/v1 
kind: Role
metadata: 
  name: developer
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","create","delete"]

$ vi dev-user-role-binding.yaml

$ cat dev-user-role-binding.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

$ kubectl apply -f dev-user-role.yaml 
role.rbac.authorization.k8s.io/developer created

$ kubectl apply -f dev-user-role-binding.yaml 
rolebinding.rbac.authorization.k8s.io/dev-user-binding created

$ kubectl get roles
NAME        CREATED AT
developer   2025-01-15T00:58:37Z

$ kubectl get rolebindings
NAME               ROLE             AGE
dev-user-binding   Role/developer   15s

$ kubectl auth can-i get pods --as dev-user
no

$ kubectl auth can-i create pods --as dev-user
yes

$ kubectl auth can-i list pods --as dev-user
yes

$ kubectl auth can-i delete pods --as dev-user
yes
```

-----

### Cluster Roles

![namespace_and_cluster_scoped_resource](namespace_and_cluster_scoped_resource.png)

```bash
$ kubectl api-resources --namespaced=true
$ kubectl api-resources --namespaced=false
```
Here's a scenario:

A new user `michelle` joined the team. She will be focusing on the `nodes` in the cluster. Create the required `ClusterRoles` and `ClusterRoleBindings` so she gets access to the nodes.

```bash
# Creating a cluster role

$ kubectl create clusterrole node-admin --verb=get,list,watch --resource=nodes
or
$ cat michelle-clusterrole.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-admin
rules:
- apiGroups:
  - '*'
  resources:
  - 'nodes'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'

$ kubectl apply -f michelle-clusterrole.yaml 
clusterrole.rbac.authorization.k8s.io/node-admin created

# Creating a cluster-role-bindings

$ kubectl create clusterrolebinding node-admin-clusterrolebinding --clusterrole=node-admin --user=michelle
or
$ cat michelle-bindings.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-admin-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: node-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: michelle

$ kubectl apply -f michelle-bindings.yaml 
clusterrolebinding.rbac.authorization.k8s.io/node-admin-rolebinding created
```

michelle's responsibilities are growing and now she will be responsible for storage as well. Create the required `ClusterRoles` and `ClusterRoleBindings` to allow her access to `Storage`.


Get the API groups and resource names from command kubectl api-resources. Use the given spec:

Required options:
- ClusterRole: storage-admin
- Resource: persistentvolumes
- Resource: storageclasses
- ClusterRoleBinding: michelle-storage-admin
- ClusterRoleBinding Subject: michelle
- ClusterRoleBinding Role: storage-admin

```bash
# Checking API Resources and shorthand
$ kubectl api-resources | grep -e "volumes"
persistentvolumes                   pv           v1                                false        PersistentVolume

$ kubectl api-resources | grep -e "storage"
csidrivers                                       storage.k8s.io/v1                 false        CSIDriver
csinodes                                         storage.k8s.io/v1                 false        CSINode
csistoragecapacities                             storage.k8s.io/v1                 true         CSIStorageCapacity
storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass
volumeattachments                                storage.k8s.io/v1                 false        VolumeAttachment

# Creating Cluster Role for Storage Admin

$ kubectl create clusterrole storage-admin --verb=* --resource=storageclasses,persistentvolumes
or
$ cat storage-role.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: storage-admin
rules:
- apiGroups: ["storage.k8s.io/v1","v1"]
  resources: ["persistentvolumes","storageclasses"]
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'

$ kubectl apply -f storage-role.yaml 
clusterrole.rbac.authorization.k8s.io/storage-admin created

# Creating cluster role bindings for the storage admin

$ kubectl create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin --user=michelle
or
$ cat bindings-storage.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: michelle-storage-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: storage-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: michelle

$ kubectl apply -f bindings-storage.yaml 
clusterrolebinding.rbac.authorization.k8s.io/michelle-storage-admin created
```

### Service Accounts

![service_account_correlation_with_ckad](service_account_correlation_with_ckad.png)

There is some correlation with the CKA and CKAD with respect to Security.

`User accounts` are used by us i.e. humans.
`Service Accounts` are used by Machines i.e. bots like `Prometheus` to collect metrics and `Jenkins`.

How do this Service Account works?

Whenever, we create a serviceaccount, let's call it - `dashboard-sa`, it will create a `service account object` and then, generates a token for the `service account object - dashboard-sa`. Post creating a token, it creates a `secret` associating the token generated. Finally, this `secret` gets associated to the `service account object - dashboard-sa`. 

We can always check this `secret token` that is generated and pass it via the `REST API` bearer token. 

```bash
curl https://192.168.56.20:6443/api --insecure --header "Authorization: Bearer <token_generated_or_used_in_secrets"
```

##### Commands used in the Service Account

```bash
$ kubectl create serviceaccount <account-name>

$ kubectl get serviceaccount

$ kubectl describe serviceaccount <account-name>

$ kubectl get sa
NAME      SECRETS   AGE
default   0         7m45s
dev       0         71s

$ kubectl get sa default -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2025-01-16T23:44:19Z"
  name: default
  namespace: default
  resourceVersion: "386"
  uid: c79eb0fb-f88b-4705-a3fc-7c6b92a369f5

$ kubectl describe sa default
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>   # there is no secret token used by the `default` service account
Events:              <none>

# Deployed a new application
$ kubectl get deploy
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
web-dashboard   1/1     1            1           44s

$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
web-dashboard-66d58fc7b8-ln9w6   1/1     Running   0          38s

$ kubectl describe pod web-dashboard-66d58fc7b8-ln9w6 
Name:             web-dashboard-66d58fc7b8-ln9w6
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.219.159
Start Time:       Thu, 16 Jan 2025 23:53:59 +0000
Labels:           name=web-dashboard
                  pod-template-hash=66d58fc7b8
Annotations:      <none>
Status:           Running
IP:               10.42.0.9
IPs:
  IP:           10.42.0.9
Controlled By:  ReplicaSet/web-dashboard-66d58fc7b8
Containers:
  web-dashboard:
    Container ID:   containerd://cc73e9f4a69b88a0a2777ce4cce30e1b43057705ddb26496b271cbe577a7649c
    Image:          gcr.io/kodekloud/customimage/my-kubernetes-dashboard
    Image ID:       gcr.io/kodekloud/customimage/my-kubernetes-dashboard@sha256:7d70abe342b13ff1c4242dc83271ad73e4eedb04e2be0dd30ae7ac8852193069
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 16 Jan 2025 23:54:02 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      PYTHONUNBUFFERED:  1
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2hjkz (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-2hjkz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  35m   default-scheduler  Successfully assigned default/web-dashboard-66d58fc7b8-ln9w6 to controlplane
  Normal  Pulling    35m   kubelet            Pulling image "gcr.io/kodekloud/customimage/my-kubernetes-dashboard"
  Normal  Pulled     35m   kubelet            Successfully pulled image "gcr.io/kodekloud/customimage/my-kubernetes-dashboard" in 1.697s (1.697s including waiting). Image size: 31659887 bytes.
  Normal  Created    35m   kubelet            Created container web-dashboard
  Normal  Started    35m   kubelet            Started container web-dashboard

$ cat /var/rbac/pod-reader-role.yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups:
  - ''
  resources:
  - pods
  verbs:
  - get
  - watch
  - list

$ kubectl describe role pod-reader
Name:         pod-reader
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get watch list]

$ cat /var/rbac/dashboard-sa-role-binding.yaml 
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: dashboard-sa # Name is case sensitive
  namespace: default
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io

$ kubectl describe rolebindings read-pods 
Name:         read-pods
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  pod-reader
Subjects:
  Kind            Name          Namespace
  ----            ----          ---------
  ServiceAccount  dashboard-sa  default

# To login or ssh into the container within the pod to check the Service Account Token is available

$ kubectl exec web-dashboard-66d58fc7b8-ln9w6 -c web-dashboard -i -t -- sh
/opt/webapp-conntest # ls
Dockerfile        app.py            requirements.txt  templates
README.md         kube-files        static            venv

$ /run/secrets/kubernetes.io/serviceaccount # ls -ltrh
total 0      
lrwxrwxrwx    1 root     root          12 Jan 16 23:54 token -> ..data/token
lrwxrwxrwx    1 root     root          16 Jan 16 23:54 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root          13 Jan 16 23:54 ca.crt -> ..data/ca.crt

$ cat token 
eyJhbGciOiJSUzI1NiIsImtpZCI6Il80WDhnTXZNcjBuUDBnV21ncWVPWm5OSmwzRnZ4VDVyVWNRbnlDSmdCRXGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxNzY4NjA3NjQwLCJpYXQiOjE3MzcXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiOWQ2OWUwMTItNjZjOS00ZjNmLWFjODktNDZyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwibm9kZSI6eyJuYW1lIjoiY29udHJvbHBsYW5lIiwidWlkIjoiM2Y5OTJjYwIn0sInBvZCI6eyJuYW1lIjoid2ViLWRhc2hib2FyZC02NmQ1OGZjN2I4LWxuOXc2IiwidWlkIjoiZDY4OWVTZkIn0sInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJkZWZhdWx0IiwidWlkIjoiYzc5ZWIwZmItZjg4Yi00NzAnRlciI6MTczNzA3NTI0N30sIm5iZiI6MTczNzA3MTY0MCwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRH_MURWc46j_HC-xzaRcAaQ9XZIufjGocWql21znwBStd2S8nQw7v5ELhNL6Y-pO6lYq8WNlVpOd9qYftrBbbPLQlvFXh5A7qZTgo__ESEFMr_OTRD1oRGt9PR5TFFlrb3Hkwmicgs2uFYcIVhBsHUZGy9tq3MllZ_3YbNsCccjK_

# There is no Service Account specifically associated to the pod or the container, it is using the default service account:
$ kubectl get deployments.apps web-dashboard -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
  ...
  name: web-dashboard
  namespace: default
  resourceVersion: "870"
  uid: f6cbf347-705a-4a14-8481-7b915b41f910
spec:
...
  selector:
    matchLabels:
      name: web-dashboard
...
  template:
    metadata:
      labels:
        name: web-dashboard
    spec:
      containers:
      - env:
        - name: PYTHONUNBUFFERED
          value: "1"
        image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
        imagePullPolicy: Always
        name: web-dashboard
        ports:
        - containerPort: 8080
          protocol: TCP
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
...
...

# Now, Edit the deployment and add the serviceAccountName: dashboard-sa, let's check the pod now:

$ kubectl get pod/web-dashboard-6f64cd9d8d-bqqhm -o yaml
apiVersion: v1
kind: Pod
metadata:
  generateName: web-dashboard-6f64cd9d8d-
  labels:
    name: web-dashboard
    pod-template-hash: 6f64cd9d8d
  name: web-dashboard-6f64cd9d8d-bqqhm
  namespace: default
...
spec:
  containers:
  - env:
    - name: PYTHONUNBUFFERED
      value: "1"
    image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
    imagePullPolicy: Always
    name: web-dashboard
    ports:
    - containerPort: 8080
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-pq2m4
      readOnly: true
  serviceAccount: dashboard-sa      # Service Account got added
  serviceAccountName: dashboard-sa
...
...
```

v1.22/1.24 - There is an update to the `Service Account` and `tokenRequest` API methods. And, how it mounted to the pod.

###### References:

KEP 1205 - Bound Service Account Tokens
https://github.com/kubernetes/enhancements/tree/master/keps/sig-auth/1205-bound-service-account-tokens

KEP-2799: Reduction of Secret-based Service Account Tokens
https://github.com/kubernetes/enhancements/tree/master/keps/sig-auth/2799-reduction-of-secret-based-service-account-token
https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets
https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

----

### Image Security

Securing images, and how to make use of the images from the `private repository` or `secured repository`.

Usually, when we have `image: nginx`, but actually this translates to `image: docker.io/library/nginx`
So,
**
- Registry -> docker.io
- User/account -> library
- Image/Repository -> nginx
**

As the `nginx` is a well-known repository or image, it is aliased as just `nginx`.

Also, for an example of the `private repository` is -> `gcr.io/kubernetes-e2e-test-images/dnsutils`
Where,
**
- Registry -> gcr.io
- User/account -> kubernetes-e2e-test-images
- Image/Repository -> dnsutils
**

In Private Repository, to access the image, we must perform the below steps:

- docker login private-registry.io
- docker run private-registry.io/apps/internal-app

But, when creating a pod, how do we do the above steps?


```bash
# We should create a secret and attach that secret to the pod

$ kubectl create secret docker-registry <secret-name> \
--docker-server=private-registry.io \
--docker-username=registry-user \
--docker-password=registry-password \
--docker-email=registry-user@org.com

# Example:

$ kubectl create secret docker-registry private-reg-cred \
> --docker-server=myprivateregistry.com:5000 \
> --docker-username=dock_user \
> --docker-password=dock_password \
> --docker-email=dock_user@myprivateregistry.com
secret/private-reg-cred created
```

```yaml
# pod-definition-file.yaml attached with the private registry

apiVersion: v1
kind: Pod
metadata: 
  name: pod-private-repo
spec:
  containers:
  - name: private-image
    image: private-registry.io/apps/internal-app
  imagePullSecrets:
  - name: <secret-name>
```

Here's a Scenario, where there is already a deployment:

```yaml
$ kubectl get deployments.apps web -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web
  name: web
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
...
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: myprivateregistry.com:5000/nginx:alpine    ## Private Registry
        imagePullPolicy: IfNotPresent
        name: nginx
...
...
```
As we have created the secret `private-reg-cred` so, we can edit the deployment and associate it in the containers:

```yaml
$ kubectl get deployments.apps web -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web
  name: web
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
...
  template:
    metadata:
      labels:
        app: web
    spec:
      imagePullSecrets:         ## Added the secrets so it can authenticate successfully
      - name: private-reg-cred
      containers:
      - image: myprivateregistry.com:5000/nginx:alpine    ## Private Registry
        imagePullPolicy: IfNotPresent
        name: nginx
...
...
```
Now, post this change the pods are `running` successfully, let's inspect those pods:

```bash
$ kubectl describe pod web-8569544bdb-m9h2g
Name:             web-8569544bdb-m9h2g
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.121.237
Start Time:       Fri, 17 Jan 2025 01:31:24 +0000
Labels:           app=web
                  pod-template-hash=8569544bdb
Annotations:      <none>
Status:           Running
IP:               10.50.0.6
IPs:
  IP:           10.50.0.6
Controlled By:  ReplicaSet/web-8569544bdb
Containers:
  nginx:
    Container ID:   containerd://48cc41c8e059a020961f940e014bab9d17b1985be50ae3c28ce18de7dd742c31
    Image:          myprivateregistry.com:5000/nginx:alpine
    Image ID:       docker.io/library/nginx@sha256:814a8e88df978ade80e584cc5b333144b9372a8e3c98872d07137dbf3b44d0e4
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 17 Jan 2025 01:31:25 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lnfrb (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-lnfrb:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m49s  default-scheduler  Successfully assigned default/web-8569544bdb-m9h2g to controlplane
  Normal  Pulling    3m48s  kubelet            Pulling image "myprivateregistry.com:5000/nginx:alpine"
  Normal  Pulled     3m48s  kubelet            Successfully pulled image "myprivateregistry.com:5000/nginx:alpine" in 119ms (119ms including waiting). Image size: 20534112 bytes.
  Normal  Created    3m48s  kubelet            Created container nginx
  Normal  Started    3m48s  kubelet            Started container nginx
```

We could clearly notice that the image was pulled from the `myprivateregistry.com:5000/nginx:alpine` as you see in the events.

------

### Pre-requisite Security in Docker


##### process isolation

Please note,  process `PID` varies between the `host` and the `docker container`. 

Let's say, you run `docker run sleep 3600`, if you notice this within the `docker container` as `ps aux`, you can see the `sleep 3600` as `PID` is `1` and the `user` is `root`.

But, when you run the same `ps aux` in the `host` terminal where the docker is installed, you would still see this `sleep 3600` but, having a different `PID` why because, both of isolated using a concept called `namespace` and `process isolation`.

By default, `host` has a namespace and the `docker` container(s) have a namespace where both does not get overlapped. At firstsight, it might look the same but, it's NOT.

That said, there is an interesting concept called `Linux Capabilities` where, we can extend the capabilities of the user in the docker i.e. `root` similar to the user in the host, also `root`. **Remember, both the `root` user(s) capabilities are different. In short, the `root` user in the docker does not have all capabilities to bypass or update or modify the data in the `host` side. So, we can extend the privileges for the `root` user in the docker container.

Here is the difference in the processes:

```bash
# Running in the host where, the command sleep 4800 is executed
$ ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 {kinit.sh} /bin/bash /usr/bin/kinit.sh
   56 root      1:00 {k3s-server} /usr/bin/k3s server
  376 root      0:15 containerd 
 ...
 ...
 8778 root      0:00 /var/lib/rancher/k3s/data/d837e0b203a84cc7bfbb4a3054fc1d95d2b9ad5e460ed34401ff795b9c03118b/bin/containerd-shim
 8804 65535     0:00 /pause
 8910 root      0:00 sleep 4800        ### Same command but, the PID is different in the host
 9608 root      0:00 /usr/bin/script -q -f /root/.terminal_logs/terminal.log bash -c sudo su -
 9610 root      0:00 -bash
 9958 root      0:00 ps aux

# Running inside the container where, the command sleep 4800 is executed, the user is same but, the PID is different
$ crictl exec -it 9cc7b88780a0d /bin/sh
# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   2696  1064 ?        Ss   13:26   0:00 sleep 4800       ### Same command but, the PID is different in the container
root          62  0.0  0.0   2800  1088 pts/0    Ss   13:29   0:00 /bin/sh
root          68  0.0  0.0   7888  4072 pts/0    R+   13:29   0:00 ps aux
# exit
 ```

##### Linux Capabilities

List of all capabilities, can be observed in the below file:

```bash
/usr/include/linux/capability.h
```

Have attached the capability file from a docker container here: [docker_container_capabilities.h.file.txt](certified-kubernetes-administrator/docker_container_capabilities.h.file.txt)

To Extend a `Capability` or use a different user instead of the `root`, you can run:

```bash
$ docker run --user=1001 ubuntu sleep 3600

$ docker run --cap-add MAC_ADMIN ubuntu sleep 3600
```

-----

### Security Contexts

Container Security in Kubernetes is same as the Docker container:

In K8s, you have an extra option to tune the `security contexts` i.e. you can spread the capabilities within the `pod` i.e. all containers in the pod will comply to those settings. Also, you can narrow it down to a `container` as well. Or, you can mix both - add `pod` level capabilities as well as the `container` level capabilities, where the `container` level capabilities are taken into consideration instead of the `pod` level capabilities because it has more priority.

```yaml
# pod-definition-file.v1.yaml

## Adding Pod-Level Security Contexts
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  securityContext:      ## Adding Pod-level security contexts
    runAsUser: 1000
  containers:
  - image: nginx
    name: nginx-app
    command: ["sleep","3600"]
```

```yaml
# pod-definition-file.v2.yaml

## Adding container-Level Security Contexts and Security Capabilities
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
    command: ["sleep","3600"]
    securityContext:      ## Adding container-level security contexts
      runAsUser: 1000
      capabilities:       ## Adding container-level capabilities
        add: ["MAC_ADMIN"]
```

>[!Important]
> Please note, the `capabilities` can only be added to the `container` and NOT the `pod` so, it will NOT work if you added the `capabilities` to the pod's security context, it will throw an error.

##### Commands used

```bash
# Post updating the user to run the `sleep 4800` command:

$ ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 {kinit.sh} /bin/bash /usr/bin/kinit.sh
   56 root      0:40 {k3s-server} /usr/bin/k3s server
  385 root      0:11 containerd 
...
...
 7033 root      0:00 /var/lib/rancher/k3s/data/d837e0b203a84cc7bfbb4a3054fc1d95d2b9ad5e460ed34401f
 7060 1010      0:00 /pause
 7165 1010      0:00 sleep 4800   ### If you see here, it got updated from `root` to user `1010`
 7308 root      0:00 ps aux

$ cat multi-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]

```

### Network Policies

#### Traffic

#### Network Security

##### Ingress Policy
```yaml
# Ingress Policy
apiVersion: netowkring.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:              # Rule 1
        matchLabels:
          name: api-pod
      namespaceSelector:        # can be used to select pods from the specific namespace
        matchLabels:
          name: prod
    - ipBlock:                  # Rule 2
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
```

>[!Caution]
> Please note, in the above `policyTypes`, we have just the `Ingress` policies and NOT the `Egress` so, here only the `Ingress` policy is regulated but, the `Egress` allows all and NO restriction for it until it is defined in the `policyTypes`.

##### Egress Policy

```yaml
# Egress Policy
apiVersion: netowkring.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:              # Rule 1
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```

>[!Important]
> Please note, in the above `policyTypes`, we have both the `Ingress` and `Egress` policy types so, both the `Incoming` and the `Outgoing` policy types are regulated.

##### Bonus Tip:

If you want to check, whether a specific component is a `array` or `string` or `integer` or `list`, then we can use this command and explain it further:

```bash
### Example 1:
$ kubectl explain networkpolicy.spec.ingress.ports
GROUP:      networking.k8s.io
KIND:       NetworkPolicy
VERSION:    v1

FIELD: ports <[]NetworkPolicyPort>

DESCRIPTION:
    ports is a list of ports which should be made accessible on the pods
    selected for this rule. Each item in this list is combined using a logical
    OR. If this field is empty or missing, this rule matches all ports (traffic
    not restricted by port). If this field is present and contains at least one
    item, then this rule allows traffic only if the traffic matches at least one
    port in the list.
    NetworkPolicyPort describes a port to allow traffic on
    
FIELDS:
  endPort       <integer>
    endPort indicates that the range of ports from port to endPort if set,
    inclusive, should be allowed by the policy. This field cannot be defined if
    the port field is not defined or if the port field is defined as a named
    (string) port. The endPort must be equal or greater than port.

  port  <IntOrString>
    port represents the port on the given protocol. This can either be a
    numerical or named port on a pod. If this field is not provided, this
    matches all port names and numbers. If present, only traffic on the
    specified protocol AND port will be matched.

  protocol      <string>
  enum: SCTP, TCP, UDP
    protocol represents the protocol (TCP, UDP, or SCTP) which traffic must
    match. If not specified, this field defaults to TCP.
    
    Possible enum values:
     - `"SCTP"` is the SCTP protocol.
     - `"TCP"` is the TCP protocol.
     - `"UDP"` is the UDP protocol.

### Example 2:

$ kubectl explain networkpolicy.spec.egress.to
GROUP:      networking.k8s.io
KIND:       NetworkPolicy
VERSION:    v1

FIELD: to <[]NetworkPolicyPeer>

DESCRIPTION:
    to is a list of destinations for outgoing traffic of pods selected for this
    rule. Items in this list are combined using a logical OR operation. If this
    field is empty or missing, this rule matches all destinations (traffic not
    restricted by destination). If this field is present and contains at least
    one item, this rule allows traffic only if the traffic matches at least one
    item in the to list.
    NetworkPolicyPeer describes a peer to allow traffic to/from. Only certain
    combinations of fields are allowed
    
FIELDS:
  ipBlock       <IPBlock>
    ipBlock defines policy on a particular IPBlock. If this field is set then
    neither of the other fields can be.

  namespaceSelector     <LabelSelector>
    namespaceSelector selects namespaces using cluster-scoped labels. This field
    follows standard label selector semantics; if present but empty, it selects
    all namespaces.
    
    If podSelector is also set, then the NetworkPolicyPeer as a whole selects
    the pods matching podSelector in the namespaces selected by
    namespaceSelector. Otherwise it selects all pods in the namespaces selected
    by namespaceSelector.

  podSelector   <LabelSelector>
    podSelector is a label selector which selects pods. This field follows
    standard label selector semantics; if present but empty, it selects all
    pods.
    
    If namespaceSelector is also set, then the NetworkPolicyPeer as a whole
    selects the pods matching podSelector in the Namespaces selected by
    NamespaceSelector. Otherwise it selects the pods matching podSelector in the
    policy's own namespace.
```

##### Commands Used:

```bash
$ kubectl api-resources
or 
$ kubectl api-resources | grep -e "network"
ingressclasses                                   networking.k8s.io/v1              false        IngressClass
ingresses                           ing          networking.k8s.io/v1              true         Ingress
networkpolicies                     netpol       networking.k8s.io/v1              true         NetworkPolicy
or 
$ kubectl api-resources | head -n 1; kubectl api-resources | grep -e "network"
or
$ kubectl api-resources | awk 'NR==1 || /network/'

$ kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
external   1/1     Running   0          6m31s
internal   1/1     Running   0          6m31s
mysql      1/1     Running   0          6m31s
payroll    1/1     Running   0          6m31s

$ kubectl get netpol
NAME             POD-SELECTOR   AGE
payroll-policy   name=payroll   8m8s

$ kubectl get netpol -A
NAMESPACE   NAME             POD-SELECTOR   AGE
default     payroll-policy   name=payroll   8m13s

$ kubectl get netpol payroll-policy -o yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.k8s.io/v1","kind":"NetworkPolicy","metadata":{"annotations":{},"name":"payroll-policy","namespace":"default"},"spec":{"ingress":[{"from":[{"podSelector":{"matchLabels":{"name":"internal"}}}],"ports":[{"port":8080,"protocol":"TCP"}]}],"podSelector":{"matchLabels":{"name":"payroll"}},"policyTypes":["Ingress"]}}
  creationTimestamp: "2025-01-17T15:16:31Z"
  generation: 1
  name: payroll-policy
  namespace: default
  resourceVersion: "1967"
  uid: a06eb044-8610-44a8-95a2-3542709ee6b0
spec:
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: internal
    ports:
    - port: 8080
      protocol: TCP
  podSelector:
    matchLabels:
      name: payroll
  policyTypes:
  - Ingress
```

>[!Important]
> **EXAM TIP**
> A good trick is to create a bash `alias` to `kubectl --dry-run=client -oyaml` and use that to generate deployment, configmap, pod, etc manifests that you can pipe to a file and tweak.
> Try performing the complete EXAM in the [killer.sh](https://killer.sh/cka)

-----

#### Scenario:

Create a network policy to allow traffic from the `Internal application` only to the `payroll-service` and `db-service`.

Use the spec given below. You might want to enable `ingress traffic` to the pod to test your rules in the UI.


```bash
Policy Name: internal-policy
Policy Type: Egress
Egress Allow: payroll
Payroll Port: 8080
Egress Allow: mysql
MySQL Port: 3306
```

![example_cluster_services_configuration](example_cluster_services_configuration.png)

#### Solution:

```yaml
# internal-policy.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector: # Applies to pods labeled as "internal"
    matchLabels:
      name: internal
  policyTypes:
    - Egress
  egress:
    # Allow traffic to "payroll" pod on port 8080 only
    - to:
        - podSelector:
            matchLabels:
              name: payroll
      ports:
        - port: 8080
          protocol: TCP
    # Allow traffic to "mysql" pod on port 3306 only
    - to:
        - podSelector:
            matchLabels:
              name: mysql
      ports:
        - port: 3306
          protocol: TCP
```

--------

### Kubectx and Kubens - Command Line Utilities:

Throughout the course, you have had to work on several different namespaces in the practice lab environments. In some labs, you also had to switch between several contexts.

While this is excellent for hands-on practice, in a real “live” Kubernetes cluster implemented for production, there could be a possibility of often switching between a large number of namespaces and clusters.

This can quickly become a confusing and overwhelming task if you have to rely on kubectl alone.

This is where command line tools such as kubectx and kubens come into the picture.

Reference:https://github.com/ahmetb/kubectx

#### Kubectx:

With this tool, you don’t have to make use of lengthy `kubectl config` commands to switch between contexts. This tool is particularly useful to switch context between clusters in a multi-cluster environment.

##### Installation:

```bash
$ sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
$ sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
```
Syntax:

To list all contexts:

```bash
$ kubectx
```

To switch to a new context:

```bash
$ kubectx
```

To switch back to the previous context:

```bash
$ kubectx –
```

To see the current context:

```bash
$ kubectx -c
```

#### Kubens:

This tool allows users to switch between namespaces quickly with a simple command.

##### Installation:

```bash
$ sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
$ sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

Syntax:

To switch to a new namespace:

```bash
$ kubens
```

To switch back to previous namespace:

```bash
kubens –
```

----------

## :: Storage

----------

### Storage Drivers - Docker Storage - File Systems

File System

When you install, it creates a folder structure:

- /var/lib/docker
  - auf
  - containers
  - image
  - volumes

COPY-ON-WRITE


volume mounting

bind mounting from any location from the docker host

Common docker storage drivers:

- AUFS
- ZFS
- BTRFS
- DEVICE_MAPPER
- OVERLAY
- OVERLAY2

Please note, the `volume drivers` are handled by the volume driver plugins.

```bash
$ docker run -it --name mysql --volume-driver rexray/ebs --mount src=ebs-vol,target=/var/lib/mysql
```

### Container Storage Interface

rocket and cri-o,

> CRI - Container Runtime Interface - CRI standards

> CNI - Container Networking Interface -> It got its standards

> CSI - Container Storage Interface -> With CSI, you write your own drivers with the CSI standards.

Here is a Github link for the `CSI standards`: https://github.com/container-storage-interface/spec and https://kubernetes-csi.github.io/docs/.

![github_csi](github_csi.png)

Note, CSI is an universal standard.

![how_csi_looks_like](how_csi_looks_like.png)

If you look at the above image:

- There are three sections: `Container Interface i.e. Kubernetes -> RPC(Remote Procedure Call) -> Storage Solution`

When the container is created, the kubernetes `should call to provision a new volume` -> `RPC` -> Then, the storage solution `should provision a new volume on the storage` and share the details back to the `kubernetes` or the `container orchestration tool`.

----

### Volumes & Mounts



Volume types

Persistent Volumes:

Access modes: `ReadOnlyMany`, `ReadWriteOnce`, and `ReadWriteMany`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:             ### Do not use this in the production scenario
    path: /tmp/data
```
Production Scenario:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

Persistent Volume Claims:

```bash
$ kubectl create -f pv-definition.yaml
$ kubectl get persistentvolume
```


-----

### Persistent Volume Claims


```yaml
# pvc-definition-file.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```bash
# To check whether the PVC is created
$ kubectl get persistentvolumeclaim
```

```yaml
# persistent-volume-definition-file.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteonce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

Now, the `persistentvolumeclaims` will get associated to the `persistentvolumes` that are available where, in this scenario, the `PV` is `1Gi` and the requirement from the `PVC` is `500Mi`.

To delete a PVC:

```bash
$ kubectl delete persistentvolumeclaim myclaim
```

>[!Important] Specifically if the `Persistent Volume` is used by the `Persistent Volume Claims` i.e. mounted to the pods. It will NOT get `DELETED` if the pod is `deleted`. Also, it depends on the `persistentVolumeReclaimPolicy: Retain` (this holds up the storage) or `persistentVolumeReclaimPolicy: Delete` (this frees up the storage) or `persistentVolumeReclaimPolicy: Recycle` (this deletes the data in the storage and make it available for the new pods or persistentvolumeclaims).

-----

### Using PVC in Pods

Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:

```yaml
# Using PVC in pods
apiVersion: v1
kind: Pod
metadata:
 name: mypod
spec:
 containers:
  - name: myfrontend
   image: nginx
   volumeMounts:
   - mountPath: "/var/www/html"
    name: mypd
 volumes:
  - name: mypd
   persistentVolumeClaim:
    claimName: myclaim
```

The same is true for `ReplicaSets or Deployments`. Add this to the pod template section of a Deployment on ReplicaSet.

Reference URL: [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

------

##### Commands Used:


```yaml
# pvc-definition-file.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
```bash
$ kubectl get pvc
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
myclaim   Pending   

$ kubectl get pvc
NAME      STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
myclaim   Bound    pv-vol1   1Gi        RWO                           <unset>                 67s

# Tried again 
$ kubectl get pv,pvc
NAME                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pv-vol1   1Gi        RWO            Retain           Bound    default/myclaim                  <unset>                          10s

NAME                            STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/myclaim   Bound    pv-vol1   1Gi        RWO                           <unset>                 5s
```

```yaml
# Using PVC in pods
apiVersion: v1
kind: Pod
metadata:
 name: mypod
spec:
 containers:
  - name: myfrontend
   image: nginx
   volumeMounts:
   - mountPath: "/log"
    name: pv-vol1
 volumes:
  - name: pv-vol1
   persistentVolumeClaim:
    claimName: myclaim
```

```yaml
$ kubectl get pod webapp -o yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-h8kx2
      readOnly: true  
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-h8kx2
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2025-01-18T23:55:27Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2025-01-18T23:55:23Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2025-01-18T23:55:27Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2025-01-18T23:55:27Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2025-01-18T23:55:23Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://be2e07d541d2b67e41df5f2726c0a8c1ecffe8d1a2231915e6e39054937e9b3d
    image: docker.io/kodekloud/event-simulator:latest
    imageID: docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    lastState: {}
    name: event-simulator
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2025-01-18T23:55:26Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-h8kx2
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 192.168.100.158
  hostIPs:
  - ip: 192.168.100.158
  phase: Running
  podIP: 172.17.0.4
  podIPs:
  - ip: 172.17.0.4
  qosClass: BestEffort
  startTime: "2025-01-18T23:55:23Z"
```

```bash
$ kubectl describe pod webapp
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.100.158
Start Time:       Sat, 18 Jan 2025 23:55:23 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.4
IPs:
  IP:  172.17.0.4
Containers:
  event-simulator:
    Container ID:   containerd://be2e07d541d2b67e41df5f2726c0a8c1ecffe8d1a2231915e6e39054937e9b3d
    Image:          kodekloud/event-simulator
    Image ID:       docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 18 Jan 2025 23:55:26 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      LOG_HANDLERS:  file
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-h8kx2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-h8kx2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m38s  default-scheduler  Successfully assigned default/webapp to controlplane
  Normal  Pulling    4m37s  kubelet            Pulling image "kodekloud/event-simulator"
  Normal  Pulled     4m35s  kubelet            Successfully pulled image "kodekloud/event-simulator" in 1.895s (1.895s including waiting). Image size: 28855042 bytes.
  Normal  Created    4m35s  kubelet            Created container event-simulator
  Normal  Started    4m35s  kubelet            Started container event-simulator
```

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:             
    path: /var/log/webapp


```bash
$ kubectl describe pod webapp
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.100.158
Start Time:       Sun, 19 Jan 2025 00:15:35 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.5
IPs:
  IP:  172.17.0.5
Containers:
  event-simulator:
    Container ID:   containerd://930865f0a3fb3964efacee60dead9374db7e0811d00da8b4c09f7ace9b8caaac
    Image:          kodekloud/event-simulator
    Image ID:       docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 19 Jan 2025 00:15:36 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      LOG_HANDLERS:  file
    Mounts:
      /log from pv-vol1 (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-h8kx2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  pv-vol1:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  myclaim
    ReadOnly:   false
  kube-api-access-h8kx2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulling  17s   kubelet  Pulling image "kodekloud/event-simulator"
  Normal  Pulled   17s   kubelet  Successfully pulled image "kodekloud/event-simulator" in 136ms (136ms including waiting). Image size: 28855042 bytes.
  Normal  Created  17s   kubelet  Created container event-simulator
  Normal  Started  16s   kubelet  Started container event-simulator
```

```yaml
cat /tmp/kubectl-edit-1546462735.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-01-18T23:55:23Z"
  name: webapp
  namespace: default
  resourceVersion: "1049"
  uid: 9ccc0f11-dcee-4fa2-911b-df8c4a7727c6
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-h8kx2
      readOnly: true
    - mountPath: /log             #### Added a volume mount
      name: pv-vol1
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: pv-vol1                  ### Added volumes
    persistentVolumeClaim:
      claimName: myclaim
  - name: kube-api-access-h8kx2
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```
```bash
$ diff --side-by-side certified-kubernetes-administrator/pod_WITHOUT_log_pv_pvc.yaml certified-kubernetes-administrator/pod_with_log_pv_pvc.yaml
apiVersion: v1                                                  apiVersion: v1
kind: Pod                                                       kind: Pod
metadata:                                                       metadata:
  name: webapp                                                    name: webapp
  namespace: default                                              namespace: default
spec:                                                           spec:
  containers:                                                     containers:
  - env:                                                          - env:
    - name: LOG_HANDLERS                                            - name: LOG_HANDLERS
      value: file                                                     value: file
    image: kodekloud/event-simulator                                image: kodekloud/event-simulator
    imagePullPolicy: Always                                         imagePullPolicy: Always
    name: event-simulator                                           name: event-simulator
                                                              >     resources: {}
    terminationMessagePath: /dev/termination-log                    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File                                  terminationMessagePolicy: File
    volumeMounts:                                                   volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccoun       - mountPath: /var/run/secrets/kubernetes.io/serviceaccoun
      name: kube-api-access-h8kx2                                     name: kube-api-access-h8kx2
      readOnly: true                                                  readOnly: true
                                                              >     - mountPath: /log             #### Added a volume mount
                                                              >       name: pv-vol1
  dnsPolicy: ClusterFirst                                         dnsPolicy: ClusterFirst
  enableServiceLinks: true                                        enableServiceLinks: true
  nodeName: controlplane                                          nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority                          preemptionPolicy: PreemptLowerPriority
  priority: 0                                                     priority: 0
  restartPolicy: Always                                           restartPolicy: Always
  schedulerName: default-scheduler                                schedulerName: default-scheduler
  securityContext: {}                                             securityContext: {}
  serviceAccount: default                                         serviceAccount: default
  serviceAccountName: default                                     serviceAccountName: default
  terminationGracePeriodSeconds: 30                               terminationGracePeriodSeconds: 30
  tolerations:                                                    tolerations:
  - effect: NoExecute                                             - effect: NoExecute
    key: node.kubernetes.io/not-ready                               key: node.kubernetes.io/not-ready
    operator: Exists                                                operator: Exists
    tolerationSeconds: 300                                          tolerationSeconds: 300
  - effect: NoExecute                                             - effect: NoExecute
    key: node.kubernetes.io/unreachable                             key: node.kubernetes.io/unreachable
    operator: Exists                                                operator: Exists
    tolerationSeconds: 300                                          tolerationSeconds: 300
  volumes:                                                        volumes:
                                                              >   - name: pv-vol1                  ### Added volumes
                                                              >     persistentVolumeClaim:
                                                              >       claimName: myclaim
  - name: kube-api-access-h8kx2                                   - name: kube-api-access-h8kx2
    projected:                                                      projected:
      defaultMode: 420                                                defaultMode: 420
      sources:                                                        sources:
      - serviceAccountToken:                                          - serviceAccountToken:
          expirationSeconds: 3607                                         expirationSeconds: 3607
          path: token                                                     path: token
      - configMap:                                                    - configMap:
          items:                                                          items:
          - key: ca.crt                                                   - key: ca.crt
            path: ca.crt                                                    path: ca.crt
          name: kube-root-ca.crt                                          name: kube-root-ca.crt
      - downwardAPI:                                                  - downwardAPI:
          items:                                                          items:
          - fieldRef:                                                     - fieldRef:
              apiVersion: v1                                                  apiVersion: v1
              fieldPath: metadata.namespace                                   fieldPath: metadata.namespace
            path: namespace
\ No newline at end of file
                                                    path: namespace
\ No newline at end of file
```

##### Added HostPath

```bash
$ kubectl describe pod webapp
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.100.158
Start Time:       Sun, 19 Jan 2025 00:40:59 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.6
IPs:
  IP:  172.17.0.6
Containers:
  event-simulator:
    Container ID:   containerd://a159c9827223182113c8cc04c2d6855d29b7b858e0b41bd1839e9172a7304481
    Image:          kodekloud/event-simulator
    Image ID:       docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 19 Jan 2025 00:41:00 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      LOG_HANDLERS:  file
    Mounts:
      /log from pv-vol2 (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-h8kx2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  pv-vol2:
    Type:          HostPath (bare host directory volume)
    Path:          /var/log/webapp
    HostPathType:  
  kube-api-access-h8kx2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulling  11s   kubelet  Pulling image "kodekloud/event-simulator"
  Normal  Pulled   11s   kubelet  Successfully pulled image "kodekloud/event-simulator" in 138ms (138ms including waiting). Image size: 28855042 bytes.
  Normal  Created  10s   kubelet  Created container event-simulator
  Normal  Started  10s   kubelet  Started container event-simulator

```

```
cat pv-file-2.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:             
    path: /pv/log

$ kubectl apply -f pv-file-2.yaml 
persistentvolume/pv-log created

$ kubectl get pv,pvc
NAME                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pv-log    100Mi      RWX            Retain           Available                          <unset>                          8s
persistentvolume/pv-vol2   1Gi        RWO            Retain           Available                          <unset>                          7m54s

$ kubectl apply -f pvc-file-2.yaml 
persistentvolumeclaim/claim-log-1 created

$ kubectl get pv,pvc
NAME                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pv-log    100Mi      RWX            Retain           Available                                        <unset>                          2m10s
persistentvolume/pv-vol2   1Gi        RWO            Retain           Bound       default/claim-log-1                  <unset>                          9m56s

NAME                                STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/claim-log-1   Bound    pv-vol2   1Gi        RWO                           <unset>                 5s

cat pvc-file-2.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

-----

### Storage Classes

![dynamic_provisioning_storage_class](dynamic_provisioning_storage_class.png)

![google_cloud_storage_provisioning_storage_class](google_cloud_storage_provisioning_storage_class.png)

![difference_in_storage_creation_replication_types](difference_in_storage_creation_replication_types.png)

##### Commands Used:

```bash
kubectl get sc
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  12m

$ kubectl describe sc local-path 
Name:                  local-path
IsDefaultClass:        Yes
Annotations:           defaultVolumeType=local,objectset.rio.cattle.io/applied=H4sIAAAAAAAA/4yRz47UMAyHXwX53JYpnamqSBxg0V4QEhJoObuJOzVN4ypxi0areXeUMqDhwJ9j8ov9xZ+fARd+ophYAhhIKhHPVE1dqlhebjUUMHFwYODTj+jBY0pQwEyKDhXBPAOGIIrKElI+Ohpw9fokfp3p82UhMODFoocCpP9KVhNpFVkqi6qeMokz4i+5fAsUy/M2gYGpSXfJVhcv3nNwr984J+GfLQLOv/5T3sb9r6K0oM2V09pTmS5JaYbipzCbrVQ5ioGUdnmcypuJco/BgMaV4FqAx5787upP3BHTCAbqrhmak21Pw9Db5tAe20MzHJuhPnUH19m2w1cOe3fMTX+bbEEd8+USZeO8XIpgIGKwI8UMuHtWQMwD8PxRPNsLGHhHnjRr2fYdvuXgOJw/iMuAL8j6KPGRY9IHCWmdKcL1ewAAAP//KQ1Ko0kCAAA,objectset.rio.cattle.io/id=,objectset.rio.cattle.io/owner-gvk=k3s.cattle.io/v1, Kind=Addon,objectset.rio.cattle.io/owner-name=local-storage,objectset.rio.cattle.io/owner-namespace=kube-system,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>

### Adding More Storage Classes:
$ kubectl get sc
NAME                        PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)        rancher.io/local-path           Delete          WaitForFirstConsumer   false                  13m
local-storage               kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  7s
portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate              false                  7s
```

The `Storage Class` called `local-storage` makes use of `VolumeBindingMode` set to `WaitForFirstConsumer`. This will delay the binding and provisioning of a `PersistentVolume` until a Pod using the `PersistentVolumeClaim` is created.

```bash
$ kubectl get pod,pv,pvc,sc
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          13s

NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/local-pv   500Mi      RWO            Retain           Bound    default/local-pvc   local-storage   <unset>                          28m

NAME                              STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/local-pvc   Bound    local-pv   500Mi      RWO            local-storage   <unset>                 18m

NAME                                                    PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/local-path (default)        rancher.io/local-path           Delete          WaitForFirstConsumer   false                  42m
storageclass.storage.k8s.io/local-storage               kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  28m
storageclass.storage.k8s.io/portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate              false                  28m

### Creation of the Storage Class

$ kubectl apply -f storage.yaml 
storageclass.storage.k8s.io/delayed-volume-sc created

$ kubectl get sc
NAME                        PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
delayed-volume-sc           kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  4s
local-path (default)        rancher.io/local-path           Delete          WaitForFirstConsumer   false                  51m
local-storage               kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  37m
portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate              false                  37m

$ cat storage.yaml 
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

----

kubectl describe pod webapp
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.163.2
Start Time:       Sun, 19 Jan 2025 23:56:02 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.4
IPs:
  IP:  172.17.0.4
Containers:
  event-simulator:
    Container ID:   containerd://8a81b368d4a3b6043313e4ff2ee9e3a62c39cfc546e5c7296016926ac9cffe56
    Image:          kodekloud/event-simulator
    Image ID:       docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 19 Jan 2025 23:56:05 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      LOG_HANDLERS:  file
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-whp7r (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-whp7r:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  37s   default-scheduler  Successfully assigned default/webapp to controlplane
  Normal  Pulling    36s   kubelet            Pulling image "kodekloud/event-simulator"
  Normal  Pulled     34s   kubelet            Successfully pulled image "kodekloud/event-simulator" in 1.989s (1.989s including waiting). Image size: 28855042 bytes.
  Normal  Created    34s   kubelet            Created container event-simulator
  Normal  Started    34s   kubelet            Started container event-simulator

----------

## :: Networking

----------

### Networking Basics: Prerequisite Switching, Routing, Gateways CNI in kubernetes


#### Switching

##### Gateway

![networking_gateway](networking_gateway.png)

```bash
$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth0@if6355: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default 
    link/ether ba:02:83:3d:92:e1 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# This command displays the Kernel's Routing Table
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         169.254.1.1     0.0.0.0         UG    0      0        0 eth0
169.254.1.1     0.0.0.0         255.255.255.255 UH    0      0        0 eth0

If you notice the above, `default` tells us that from the gateway `169.254.1.1`, it will send all traffic to the router, it is either 0.0.0.0 or default. Usually, the default or 0.0.0.0 is used to reach the internet. But, if there is no default, then it is a private network routing table and you would need to add routing table config to reach the internet.

# To add a route from `Server B` to `Server C`
$ ip route add <destination_ip_address/subnet-range> via <gateway_address>

# Example:
$ ip route add 192.168.2.0/24 via 192.168.1.1

# Where, the 192.168.1.1 is the gateway of the existing VM and it want's to reach the destination of the IP/subnet range: 192.168.2.0/24. and, it is the vice-versa for all other servers.

```
#### Default Gateway

![networking_gateway_2](networking_gateway_2.png)

```bash
# added an IP route
$ ip route add 192.168.2.0/24 via 169.254.1.1

# Now, the routing table looks like:
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         169.254.1.1     0.0.0.0         UG    0      0        0 eth0
169.254.1.1     0.0.0.0         255.255.255.255 UH    0      0        0 eth0
192.168.2.0     169.254.1.1     255.255.255.0   UG    0      0        0 eth0

Please note, here the `gateway` is `169.254.1.1`. Otherwise, it will throw an error -> Error: Nexthop has invalid gateway.
```
![networking_gateway_3](networking_gateway_3.png)

>[!Important]
> Having a gateway configured, will NOT just send the packets from one interface like `eth0` to `eth1` automatically so, we need to configure it manually by `ip_forward` so that, the packets from `server A` in `private network` can travel between `server B` as a gateway to reach the `server C` in a different network.

```bash
# Make sure to pass the packets from `server A` to `server C`, we need to add the below changes in the `server B`
$ cat /proc/sys/net/ipv4/ip_forward
0

$ echo 1 > /proc/sys/net/ipv4/ip_forward
1

# To persist the changes across reboots, update the below file
$ /etc/sysctl.conf
...
net.ipv4.ip_forward = 1
...
```

##### Commands Used:

```bash
$ ip link
$ ip addr
$ ip addr add 192.168.1.10/24 dev eth0
$ ip route or route
$ ip route add 192.168.1.0/24 via 192.168.2.1
$ cat /proc/sys/net/ipv4/ip_forward
```

----

### DNS Configuration

![networking_DNS_config_1](networking_DNS_config_1.png)

```bash
$ cat /etc/hosts

# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
192.168.121.203 ubuntu-host

# Entries added by HostAliases.
10.0.0.6        docker-registry-mirror.kodekloud.com
```

When, the hosts file gets bigger and the list gets biggers, this is when we use the DNS server. We push all the hosts and its IP address to the DNS server and point all our `servers` in the network to the DNS server.

```bash
$ cat /etc/resolv.conf
search ztp6ekkbov7cfqmv.svc.cluster.local svc.cluster.local cluster.local us-central1-a.c.kk-lab-prod.internal c.kk-lab-prod.internal google.internal
nameserver 10.96.0.10
options ndots:5
```

```mermaid
sequenceDiagram
participant User Request(Terminal)
participant Server A
participant /etc/hosts
participant /etc/resolv.conf
participant DNS Server

User Request(Terminal)->>Server A: Request `$ ping db`
Server A->>/etc/hosts: Check for db(hostname) /etc/hosts file
alt Hostname found (169.29.2.4 db)
    /etc/hosts-->>Server A: Return IP address and response
else Hostname not found (no entries of db in /etc/hosts)
    /etc/hosts->>/etc/resolv.conf: Refer to public nameserver 0.0.0.0 or private nameserver 182.27.23.5
    /etc/resolv.conf->>DNS Server: Query DNS server for hostname & IP
    DNS Server-->>/etc/resolv.conf: Return IP address and response
    /etc/resolv.conf-->>Server A: Forward IP address and response
end
Server A-->>User Request(Terminal): Respond with IP address and response
```

```bash
# Order at which the DNS name resolution takes place
$ cat /etc/nsswitch.conf 
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files
group:          files
shadow:         files
gshadow:        files

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

![networking_DNS_config_2.png](networking_DNS_config_2.png)

Where, the Record:
```
A Record -> IPv4 address
AAAA Record -> IPv6 address
CNAME Record -> other names i.e. name-to-name mapping
```

##### Command Used:

```bash
# Query the DNS Servers, and will NOT be useful when it is a private network or do not query the /etc/hosts file.
$ nslookup google.com
Server:         10.96.0.10
Address:        10.96.0.10#53

Non-authoritative answer:
Name:   google.com
Address: 64.233.179.113
Name:   google.com
Address: 64.233.179.100
Name:   google.com
Address: 64.233.179.101
Name:   google.com
Address: 64.233.179.138
Name:   google.com
Address: 64.233.179.139
Name:   google.com
Address: 64.233.179.102
Name:   google.com
Address: 2607:f8b0:4001:c1d::66
Name:   google.com
Address: 2607:f8b0:4001:c1d::65
Name:   google.com
Address: 2607:f8b0:4001:c1d::8b
Name:   google.com
Address: 2607:f8b0:4001:c1d::8a

# dig is more detailed than nslookup command
$ dig google.com

; <<>> DiG 9.18.1-1ubuntu1.3-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9032
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: bd2e828ba6ac71bf (echoed)
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             30      IN      A       142.251.184.102
google.com.             30      IN      A       142.251.184.138
google.com.             30      IN      A       142.251.184.139
google.com.             30      IN      A       142.251.184.113
google.com.             30      IN      A       142.251.184.101
google.com.             30      IN      A       142.251.184.100

;; Query time: 4 msec
;; SERVER: 10.96.0.10#53(10.96.0.10) (UDP)
;; WHEN: Tue Jan 21 02:20:58 UTC 2025
;; MSG SIZE  rcvd: 207
```

-----

#### :: Using COREDNS as our DNS Server

-----

In the previous lecture, we saw why you need a DNS server, how it can help manage name resolution in large environments with many hostnames and Ips, and how you can configure your hosts to point to a DNS server. In this article, we will see how to configure a host as a DNS server.

We are given a server dedicated as the DNS server and a set of IPs to configure as entries in the server. There are many DNS server solutions out there; in this lecture, we will focus on a particular one – CoreDNS.

So, how do you get **core DNS**? CoreDNS binaries can be downloaded from their Github releases page or as a docker image. Let’s go the traditional route. Download the binary using curl or wget. And extract it. You get the coredns executable.

![installation of coredns](coredns_1.png)

* Run the executable to start a DNS server. It, by default, listens on `port 53`, which is the default port for a DNS server.

* Now, we haven’t specified the IP to hostname mappings. For that, you need to provide some configurations. There are multiple ways to do that. We will look at one.

* First, we put all of the entries into the DNS servers /etc/hosts file. Then, we configure CoreDNS to use that file. CoreDNS loads its configuration from a file named Corefile.

Here is a `simple configuration` that instructs **CoreDNS** to fetch the IP to hostname mappings from the file `/etc/hosts`. When the DNS server is run, it now picks the IPs and names from the /etc/hosts file on the server.

![configuring IP names](coredns_2.png)

**CoreDNS** also supports other ways of configuring DNS entries through plugins. We will look at the plugin that it uses for Kubernetes in a later section.

Read more about `CoreDNS` here:

* https://github.com/kubernetes/dns/blob/master/docs/specification.md
* https://coredns.io/plugins/kubernetes/

-----

### Network Namespaces

```bash
$ ip netns add red

$ ip netns add blue

$ ip netns
blue
red
cni-78909532-a9bb-7431-f0f5-a88282c6a7e9 (id: 2)
cni-e7297641-1074-288e-3b31-c16d1527fab5 (id: 1)

$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth0@if7767: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default 
    link/ether e2:66:e5:28:81:05 brd ff:ff:ff:ff:ff:ff link-netnsid 0
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether fe:69:40:36:84:ad brd ff:ff:ff:ff:ff:ff
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ce:67:ca:c2:f0:9e brd ff:ff:ff:ff:ff:ff
6: veth21def229@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue master cni0 state UP mode DEFAULT group default qlen 1000
    link/ether c2:2c:6e:e3:1a:25 brd ff:ff:ff:ff:ff:ff link-netns cni-e7297641-1074-288e-3b31-c16d1527fab5
7: veth936a0bd9@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue master cni0 state UP mode DEFAULT group default qlen 1000
    link/ether c2:00:59:e6:57:e4 brd ff:ff:ff:ff:ff:ff link-netns cni-78909532-a9bb-7431-f0f5-a88282c6a7e9

$ ip netns exec red ip link 
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

$ ip -n red link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

$ arp
Address                  HWtype  HWaddress           Flags Mask            Iface
169.254.1.1              ether   ee:ee:ee:ee:ee:ee   C                     eth0
172.17.2.0               ether   f6:54:f8:d0:9a:6b   CM                    flannel.1
172.17.0.2               ether   fa:df:b5:0e:c4:9b   C                     cni0
172.17.1.0               ether   1a:74:93:25:24:8a   CM                    flannel.1
172.17.0.3               ether   ca:ff:32:f1:83:81   C                     cni0

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         169.254.1.1     0.0.0.0         UG    0      0        0 eth0
169.254.1.1     0.0.0.0         255.255.255.255 UH    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0
172.17.1.0      172.17.1.0      255.255.255.0   UG    0      0        0 flannel.1
172.17.2.0      172.17.2.0      255.255.255.0   UG    0      0        0 flannel.1


##### Inside the namespaces:

$ ip netns exec red arp

$ ip netns exec red route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface

$ ip link add veth-red type veth peer name veth-blue

### Important Step, make use of the subnet /24 instead of /32
$ ip netns exec red ip addr add 192.168.15.1/24 dev veth-red

$ ip netns exec blue ip addr add 192.168.15.2/24 dev veth-blue

$ ip netns exec red ping 192.168.15.1
PING 192.168.15.1 (192.168.15.1) 56(84) bytes of data.

--- 192.168.15.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2026ms


$ ip netns exec red ping 192.168.15.2
PING 192.168.15.2 (192.168.15.2) 56(84) bytes of data.
64 bytes from 192.168.15.2: icmp_seq=1 ttl=64 time=0.051 ms
64 bytes from 192.168.15.2: icmp_seq=2 ttl=64 time=0.025 ms
64 bytes from 192.168.15.2: icmp_seq=3 ttl=64 time=0.027 ms
64 bytes from 192.168.15.2: icmp_seq=4 ttl=64 time=0.040 ms

--- 192.168.15.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3056ms
rtt min/avg/max/mdev = 0.025/0.035/0.051/0.010 ms


$ ip netns exec blue ping 192.168.15.1
PING 192.168.15.1 (192.168.15.1) 56(84) bytes of data.
64 bytes from 192.168.15.1: icmp_seq=1 ttl=64 time=0.046 ms
64 bytes from 192.168.15.1: icmp_seq=2 ttl=64 time=0.027 ms
64 bytes from 192.168.15.1: icmp_seq=3 ttl=64 time=0.029 ms

--- 192.168.15.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2030ms
rtt min/avg/max/mdev = 0.027/0.034/0.046/0.008 ms

$ ip netns exec red arp
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.15.2             ether   1a:b6:8f:68:38:6b   C                     veth-red

$ ip netns exec blue arp
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.15.1             ether   d2:85:a6:21:b2:a7   C                     veth-blue

$ ip netns exec blue route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.15.0    0.0.0.0         255.255.255.0   U     0      0        0 veth-blue

$ ip netns exec red route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.15.0    0.0.0.0         255.255.255.0   U     0      0        0 veth-red
```

Adding more network namespaces, it is impossible to add `peers` to one-by-one so, we can use `Switch` i.e. `Virtual Switch`.

```bash
$ ip link add v-net-0 type bridge

$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth0@if7767: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default 
    link/ether e2:66:e5:28:81:05 brd ff:ff:ff:ff:ff:ff link-netnsid 0
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether fe:69:40:36:84:ad brd ff:ff:ff:ff:ff:ff
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ce:67:ca:c2:f0:9e brd ff:ff:ff:ff:ff:ff
6: veth21def229@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue master cni0 state UP mode DEFAULT group default qlen 1000
    link/ether c2:2c:6e:e3:1a:25 brd ff:ff:ff:ff:ff:ff link-netns cni-e7297641-1074-288e-3b31-c16d1527fab5
7: veth936a0bd9@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue master cni0 state UP mode DEFAULT group default qlen 1000
    link/ether c2:00:59:e6:57:e4 brd ff:ff:ff:ff:ff:ff link-netns cni-78909532-a9bb-7431-f0f5-a88282c6a7e9

### Virtual Switch ###
10: v-net-0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether e2:eb:fa:12:d3:46 brd ff:ff:ff:ff:ff:ff

$ ip link set dev v-net-0 up

---

# Delete the veth interfaces so we can attach it to the bridge i.e. Virtual Switch created above:

$ ip -n red link del veth-red

```

>[!Important]
> Please note, whenever you delete one-peer, it will automatically deletes the other as well. In this scenario, once you perform the `dev veth-red` it will delete the `veth-blue` as well.

#### Linux Bridge

As we have a switch in place, we can connect all the network namespaces interfaces to the bridge so, it can connect to all.

```bash
### Create the cables from the network namespaces to the bridge network
$ ip link add veth-red type veth peer name veth-red-br

$ ip link add veth-blue type veth peer name veth-blue-br

### This above commands will attach the network namespaces and the other end to the bridge network

# Let's create a bridge network
$ ip link add v-net-0 type bridge

# Let's switch it ON
$ ip link set dev v-net-0 up

-----

# Let's add it for the red namespace
$ ip link set veth-red netns red

# Quick check:
$ ip netns exec red ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
12: veth-red@if11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether be:26:41:9b:fb:88 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# Let's add it for the bridge as the master
$ ip link set veth-red-br master v-net-0

----- 

# Let's follow the same for the blue namespace
$ ip link set veth-blue netns blue

# Quick check:
$ ip netns exec blue ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
14: veth-blue@if13: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 12:3c:ef:61:62:37 brd ff:ff:ff:ff:ff:ff link-netnsid 1

# Let's add it for the bridge as the master
$ ip link set veth-blue-br master v-net-0

------

# Finally, let's add the unique IP address to those network namespaces

$ ip -n red addr add 192.168.15.1/24 dev veth-red; ip -n blue addr add 192.168.15.2/24 dev veth-blue

$ ip netns exec blue ip addr; ip netns exec red ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
14: veth-blue@if13: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
    link/ether 12:3c:ef:61:62:37 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 192.168.15.2/24 scope global veth-blue
       valid_lft forever preferred_lft forever
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
12: veth-red@if11: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
    link/ether be:26:41:9b:fb:88 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.15.1/24 scope global veth-red
       valid_lft forever preferred_lft forever

# Another method:
$ ip netns exec red ip addr | grep inet; ip netns exec blue ip addr | grep inet
    inet 192.168.15.1/24 scope global veth-red
    inet 192.168.15.2/24 scope global veth-blue

--- 

# Now, let's start the veth interface

$ ip -n red link set veth-red up

$ ip -n blue link set veth-blue up

# Let's try to ping the blue network namespace from the red network namespace:

controlplane ~ ➜  ip netns exec red ping 192.168.15.2
PING 192.168.15.2 (192.168.15.2) 56(84) bytes of data.
^C
--- 192.168.15.2 ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4107ms

## Seems like it did not work, let's see whether the switches/bridge are activated or UP.

$ bridge link show
6: veth21def229@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
7: veth936a0bd9@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
11: veth-red-br@if12: <BROADCAST,MULTICAST> mtu 1500 master v-net-0 state disabled priority 32 cost 2 
13: veth-blu-br@if14: <BROADCAST,MULTICAST> mtu 1500 master v-net-0 state disabled priority 32 cost 2 

$ bridge link show master v-net-0
6: veth21def229@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
7: veth936a0bd9@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
11: veth-red-br@if12: <BROADCAST,MULTICAST> mtu 1500 master v-net-0 state disabled priority 32 cost 2 
13: veth-blu-br@if14: <BROADCAST,MULTICAST> mtu 1500 master v-net-0 state disabled priority 32 cost 2 

$ ip link set veth-red-br up; ip link set veth-blu-br up

$ bridge link show master v-net-0
6: veth21def229@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
7: veth936a0bd9@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
11: veth-red-br@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master v-net-0 state forwarding priority 32 cost 2 
13: veth-blu-br@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master v-net-0 state forwarding priority 32 cost 2 

$ ip link set v-net-0 up

$ bridge link show
6: veth21def229@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
7: veth936a0bd9@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 master cni0 state forwarding priority 32 cost 2 
11: veth-red-br@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master v-net-0 state forwarding priority 32 cost 2 
13: veth-blu-br@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master v-net-0 state forwarding priority 32 cost 2

# Now, all the interfaces are UP and FORWARDING

# Let's try to reach the blue network namespace again:

$ ip netns exec red ping 192.168.15.2
PING 192.168.15.2 (192.168.15.2) 56(84) bytes of data.
64 bytes from 192.168.15.2: icmp_seq=1 ttl=64 time=0.100 ms
64 bytes from 192.168.15.2: icmp_seq=2 ttl=64 time=0.050 ms
64 bytes from 192.168.15.2: icmp_seq=3 ttl=64 time=0.050 ms

--- 192.168.15.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2034ms
rtt min/avg/max/mdev = 0.050/0.066/0.100/0.023 ms
```

##### commands used

```bash
Commands:
$ ip netns - This lists the network namespaces
$ ip link - This shows the network interfaces and its states
$ ip addr - This shows the network interfaces and its IP addresses i.e. inet.

# Step 1: Added a new red and blue network namespace

$ ip netns add red; ip netns add blue

# Step 2: Let's create that bridge network, i.e. Switch

$ ip link add v-net-0 type bridge

# Step 3: Let's create the cables and interfaces

$ ip link add veth-red type veth peer name veth-red-br; ip link add veth-blue type veth peer name veth-blue-br

Interfaces:
veth-red => red
veth-blue => blue
veth-blue-br => bridge (port 1)
veth-red-br => bridge (port 2)

# Once adding the interfaces, i.e. the cables, let's set those cables to those interfaces:

$ ip link set veth-blue netns blue; ip link set veth-red netns red

$ ip link set veth-blue-br master v-net-0; ip link set veth-red-br master v-net-0

# Step 4: Let's add those IP addresses:

$ ip netns exec red ip addr add 192.168.15.1/24 dev veth-red;ip netns exec blue ip addr add 192.168.15.2/24 dev veth-blue

# Step 5: Let's UP all the interfaces

$ ip netns exec red ip link set veth-red up; ip netns exec blue ip link set veth-blue up
$ ip link set veth-red-br up; ip link set veth-blue-br up
$ ip link set dev v-net-0 up


# Here, you see, the veth-red interface, is DOWN "state DOWN" and once we start UP the interfaces, you see the state is UP "state UP".

$ ip netns exec red ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
10: veth-red@if9: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 9e:12:30:ef:55:70 brd ff:ff:ff:ff:ff:ff link-netnsid 0

$ ip netns exec red ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
10: veth-red@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 9e:12:30:ef:55:70 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

```mermaid
sequenceDiagram
    participant User
    participant RedNamespace
    participant BlueNamespace
    participant Bridge as Bridge (v-net-0)

    User->>OS: Add "red" namespace
    User->>OS: Add "blue" namespace

    User->>OS: Create bridge network "v-net-0"

    User->>OS: Create veth pair (veth-red, veth-red-br)
    User->>OS: Create veth pair (veth-blue, veth-blue-br)

    User->>RedNamespace: Assign veth-red to "red" namespace
    User->>BlueNamespace: Assign veth-blue to "blue" namespace

    User->>Bridge: Assign veth-red-br as port 1
    User->>Bridge: Assign veth-blue-br as port 2

    User->>RedNamespace: Assign IP 192.168.15.1/24 to veth-red
    User->>BlueNamespace: Assign IP 192.168.15.2/24 to veth-blue

    User->>RedNamespace: Bring veth-red UP
    User->>BlueNamespace: Bring veth-blue UP

    User->>Bridge: Bring veth-red-br UP
    User->>Bridge: Bring veth-blue-br UP
    User->>Bridge: Bring bridge "v-net-0" UP
```

### Internal to External Communication from the Network Namespaces

Now, let's try to add an IP address to the `v-net-0` interface i.e. the `bridge` does not have an `IP address`:

```bash
$ ip addr add 192.168.15.5/24 dev v-net-0

$ ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5

or
$ ip netns exec blue ip route add 192.168.121.0/24 via 192.168.15.5

or

$ ip netns exec blue ip route add <host_ip_address>/24 via 192.168.15.5

iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE

$ ip netns exec blue route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.15.0    0.0.0.0         255.255.255.0   U     0      0        0 veth-blue
192.168.121.0   192.168.15.5    255.255.255.0   UG    0      0        0 veth-blue

$ ip netns exec blue ip route add default via 192.168.15.5

$ ip netns exec blue route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.15.5    0.0.0.0         UG    0      0        0 veth-blue
192.168.15.0    0.0.0.0         255.255.255.0   U     0      0        0 veth-blue
192.168.121.0   192.168.15.5    255.255.255.0   UG    0      0        0 veth-blue

$ ip netns exec blue ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=0.843 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=0.216 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1012ms
rtt min/avg/max/mdev = 0.216/0.529/0.843/0.313 ms
```

### External to Internal Communication to the Network Namespaces

We can make use of `PORTFORWARDING`:

```bash
$ iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
```

Now, whatever traffic comes to the 192.168.15.2:80 in the localhost, it should be forwarded to the blue namespace.

```bash
# check IP tables

$ iptables -t nat -L POSTROUTING -n -v

$ iptables -L -n -v
$ iptables -L -n --line-numbers
```

----

### Docker Networking

To all that, we saw in the [Linux Bridge](#linux-bridge), the docker does this seemlessly taking hands-off approach. Because, it creates a `bridge` network and takes care of the interfaces/IP addresses settings.

So, whatever, we saw above, `Docker` makes it available for us to utilize:

```bash
$ docker run --name nginx-container -p 8080:80 --network bridge nginx
```

So, if you see here, this nginx container gets attached to the network `bridge` automatically which is already provisioned(by default) in the Docker network setup. Once, you make the `port forwarding` accessible through the `docker host`(as it automatically performs a NAT Network Address Translation), the `end-users` can reach via the `docker host` to the `nginx`.

Default bridge network address: `172.17.0.0` so, whenever new applications get added to this bridge network, it assigns the IP `172.17.0.1` and so on.

![docker_networking](docker_networking.png)

----

### CNI - Container Networking Interface


### Cluster Networking - Networking in Kubernetes

![open_port_k8s](open_port_k8s.png)

>[!Important]
>This table outlines the important ports and services for Kubernetes master and worker nodes.

| **Component**              | **Port(s)**           | **Description**                         |
|----------------------------|-----------------------|-----------------------------------------|
| **Master Node**            |                       |                                         |
| ETCD                       | 2379                 | Communication between ETCD nodes.       |
| kube-api                   | 6443                 | Kubernetes API server.                  |
| kubelet                    | 10250                | Kubernetes Kubelet on the master node.  |
| kube-scheduler             | 10259                | Kubernetes Scheduler.                   |
| kube-controller-manager    | 10257                | Kubernetes Controller Manager.          |
| ETCD client                | 2380 (if multi-master)| ETCD inter-node communication.          |
| **Worker Node**            |                       |                                         |
| Services                   | 30000-32767          | NodePort service range.                 |
| kubelet                    | 10250                | Kubernetes Kubelet on the worker node.  |
| kube-proxy                 | 10256                | Kubernetes kube-proxy on the worker node.  |

- Refer to this, official K8s documentation on [Ports and Protocols](https://kubernetes.io/docs/reference/networking/ports-and-protocols/).

----

##### Commands Used:

```
$ ip link 
$ ip addr
$ ip addr add 192.168.1.10/24 dev eth0
$ ip route
$ ip route add 192.168.1.0/24 via 192.168.2.1
$ cat /proc/sys/net/ipv4/ip_forward
$ arp
$ netstat -plnt
```

##### Exam Tip

An important tip about deploying `Network Addons` in a `Kubernetes cluster`.

In the upcoming labs, we will work with Network Addons. This includes installing a network plugin in the cluster. While we have used weave-net as an example, please bear in mind that you can use any of the plugins which are described here:

https://kubernetes.io/docs/concepts/cluster-administration/addons/

https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model

In the `CKA exam`, for a question that requires you to deploy a network addon, unless specifically directed, you may use any of the solutions described in the link above.

However, the documentation currently does not contain a direct reference to the exact command to be used to deploy a third-party network addon.

The links above redirect to third-party/ vendor sites or GitHub repositories, which cannot be used in the exam. This has been intentionally done to keep the content in the Kubernetes documentation vendor-neutral.

Note: In the official exam, all essential CNI deployment details will be provided

-----
##### Important K8s Networking Questions:

<details><summary>Q1:  What is the port the kube-scheduler is listening on in the controlplane node?</summary>

A: 
```bash
$ netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 192.168.233.136:2380    0.0.0.0:*               LISTEN      3367/etcd           
tcp        0      0 192.168.233.136:2379    0.0.0.0:*               LISTEN      3367/etcd           
tcp        0      0 127.0.0.1:34881         0.0.0.0:*               LISTEN      961/containerd      
tcp        0      0 127.0.0.1:2381          0.0.0.0:*               LISTEN      3367/etcd           
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      3367/etcd           
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      557/systemd-resolve 
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      3374/kube-scheduler 
tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      3903/kube-controlle 
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      4164/kubelet        
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      4631/kube-proxy     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      963/sshd: /usr/sbin 
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      965/ttyd            
tcp6       0      0 :::22                   :::*                    LISTEN      963/sshd: /usr/sbin 
tcp6       0      0 :::6443                 :::*                    LISTEN      3680/kube-apiserver 
tcp6       0      0 :::10250                :::*                    LISTEN      4164/kubelet        
tcp6       0      0 :::10256                :::*                    LISTEN      4631/kube-proxy     
tcp6       0      0 :::8888                 :::*                    LISTEN      4325/kubectl        
udp        0      0 127.0.0.53:53           0.0.0.0:*                           557/systemd-resolve 
udp        0      0 0.0.0.0:8472            0.0.0.0:*                           -                   

```
</details>

<details><summary>Q2: How many client connections are Established under ETCD and what port?
</summary>

A:
```bash
$ netstat -tulpna | head -n 1; netstat -tulpna | grep -e "etcd"
Active Internet connections (servers and established)
tcp        0      0 192.168.233.136:2380    0.0.0.0:*               LISTEN      3367/etcd           
tcp        0      0 192.168.233.136:2379    0.0.0.0:*               LISTEN      3367/etcd           
tcp        0      0 127.0.0.1:2381          0.0.0.0:*               LISTEN      3367/etcd           
...
...           
tcp        0      0 127.0.0.1:2379          127.0.0.1:48810         ESTABLISHED 3367/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:49052         ESTABLISHED 3367/etcd           
```

The ETCD client connections are established under the `port 2379` and it is the most number of connections.
</details>

<details><summary>Q3: How do you find the bridge interface among multiple interfaces?
</summary>

A: 
```bash
# show type bridge shows the interface

$ ip address show type bridge
```

</details>


-----

### Pod Networking


-----

### CNI in Kubernetes 


------

>[!Warning]
>Weave Net Installation method is updated:

Important Update:

Before going to the CNI weave lecture, we have an update for the Weave Net installation link. They have announced the end of service for Weave Cloud.

To know more about this, read the blog from the link below: –

https://www.weave.works/blog/weave-cloud-end-of-service

As an impact, the old weave net installation link won’t work anymore: –

```bash
$ kubectl apply -f “https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d ‘\n’)”
```

Instead of that, use the latest link below to install the weave net: –

```bash
$ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Reference links:

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation
https://github.com/weaveworks/weave/releases

-----

### Weave Works - Weave CNI Plugin

![weaveworks_agent_1](weaveworks_agent_1.png)

#### Deploy Weave

Can be deployed as `Services` or `DaemonSets`, or we can deploy it as `Pods`:

```bash
# Latest Installation binary to deploy Weavenet:

$ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

```bash
# To Troubleshoot the weave related issues:
$ kubectl get pods -n kube-system

$ kubectl logs weave-net-<id> weave -n kube-system
```
-----

##### Commands Used:

**How to find the config of the `kubelet service`?**

```bash
$ systemctl status kubelet 
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Fri 2025-01-24 04:56:02 UTC; 17min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 4144 (kubelet)
      Tasks: 23 (limit: 77143)
     Memory: 42.3M
     CGroup: /system.slice/kubelet.service
             └─4144 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.10

Jan 24 04:56:10 controlplane kubelet[4144]: E0124 04:56:10.478671    4144 log.go:32] "RunPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"593e62ae3de97ba7f64f64e8b3903ee8706950dba749f92b9cbe91fc7415ea3b\": plugin type=\"flannel\" failed (add): loadFlannelSubnetEnv failed: open /run/flannel/subnet.env: no such file or directory"
Jan 24 04:56:10 controlplane kubelet[4144]: E0124 04:56:10.478719    4144 kuberuntime_sandbox.go:72] "Failed to create sandbox for pod" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"593e62ae3de97ba7f64f64e8b3903ee8706950dba749f92b9cbe91fc7415ea3b\": plugin type=\"flannel\" failed (add): loadFlannelSubnetEnv failed: open /run/flannel/subnet.env: no such file or directory" pod="kube-system/coredns-77d6fd4654-ghwd2"
Jan 24 04:56:10 controlplane kubelet[4144]: E0124 04:56:10.478739    4144 kuberuntime_manager.go:1168] "CreatePodSandbox for pod failed" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"593e62ae3de97ba7f64f64e8b3903ee8706950dba749f92b9cbe91fc7415ea3b\": plugin type=\"flannel\" failed (add): loadFlannelSubnetEnv failed: open /run/flannel/subnet.env: no such file or directory" pod="kube-system/coredns-77d6fd4654-ghwd2"
Jan 24 04:56:10 controlplane kubelet[4144]: E0124 04:56:10.478782    4144 pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"coredns-77d6fd4654-ghwd2_kube-system(382938d7-b72b-45ed-be37-7ea0ccef7e1b)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"coredns-77d6fd4654-ghwd2_kube-system(382938d7-b72b-45ed-be37-7ea0ccef7e1b)\\\": rpc error: code = Unknown desc = failed to setup network for sandbox \\\"593e62ae3de97ba7f64f64e8b3903ee8706950dba749f92b9cbe91fc7415ea3b\\\": plugin type=\\\"flannel\\\" failed (add): loadFlannelSubnetEnv failed: open /run/flannel/subnet.env: no such file or directory\"" pod="kube-system/coredns-77d6fd4654-ghwd2" podUID="382938d7-b72b-45ed-be37-7ea0ccef7e1b"
Jan 24 04:56:11 controlplane kubelet[4144]: I0124 04:56:11.857913    4144 pod_startup_latency_tracker.go:104] "Observed pod startup duration" pod="kube-flannel/kube-flannel-ds-s5l4d" podStartSLOduration=4.857894787 podStartE2EDuration="4.857894787s" podCreationTimestamp="2025-01-24 04:56:07 +0000 UTC" firstStartedPulling="0001-01-01 00:00:00 +0000 UTC" lastFinishedPulling="0001-01-01 00:00:00 +0000 UTC" observedRunningTime="2025-01-24 04:56:11.857736072 +0000 UTC m=+9.173816404" watchObservedRunningTime="2025-01-24 04:56:11.857894787 +0000 UTC m=+9.173975117"
Jan 24 04:56:25 controlplane kubelet[4144]: I0124 04:56:25.878677    4144 pod_startup_latency_tracker.go:104] "Observed pod startup duration" pod="kube-system/coredns-77d6fd4654-6txv9" podStartSLOduration=18.878401825 podStartE2EDuration="18.878401825s" podCreationTimestamp="2025-01-24 04:56:07 +0000 UTC" firstStartedPulling="0001-01-01 00:00:00 +0000 UTC" lastFinishedPulling="0001-01-01 00:00:00 +0000 UTC" observedRunningTime="2025-01-24 04:56:25.878106912 +0000 UTC m=+23.194187237" watchObservedRunningTime="2025-01-24 04:56:25.878401825 +0000 UTC m=+23.194482158"
Jan 24 04:56:26 controlplane kubelet[4144]: I0124 04:56:26.893688    4144 pod_startup_latency_tracker.go:104] "Observed pod startup duration" pod="kube-system/coredns-77d6fd4654-ghwd2" podStartSLOduration=19.893648614 podStartE2EDuration="19.893648614s" podCreationTimestamp="2025-01-24 04:56:07 +0000 UTC" firstStartedPulling="0001-01-01 00:00:00 +0000 UTC" lastFinishedPulling="0001-01-01 00:00:00 +0000 UTC" observedRunningTime="2025-01-24 04:56:26.883350491 +0000 UTC m=+24.199430814" watchObservedRunningTime="2025-01-24 04:56:26.893648614 +0000 UTC m=+24.209728944"
Jan 24 05:01:02 controlplane kubelet[4144]: E0124 05:01:02.841875    4144 info.go:104] Failed to get disk map: could not parse device numbers from  for device md127
Jan 24 05:06:02 controlplane kubelet[4144]: E0124 05:06:02.842301    4144 info.go:104] Failed to get disk map: could not parse device numbers from  for device md127
Jan 24 05:11:02 controlplane kubelet[4144]: E0124 05:11:02.842568    4144 info.go:104] Failed to get disk map: could not parse device numbers from  for device md127
```

**Or Alternative methods:**

```bash
$ ps -aux | grep -i kubelet | grep container-runtime
root        4212  0.0  0.0 4188260 75976 ?       Ssl  11:52   0:41 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.10
```

##### CNI Questions:

<details><summary>Q1: What is the path configured with all binaries of CNI supported plugins?</summary>

```bash
$ /opt/cni/bin
```
</details>

<details><summary>Q2: Identify which of the below plugins is not available in the list of available CNI plugins on this host?</summary>

```bash
$ ls  -ltrh /opt/cni/bin/
total 82M
4.0M -rwxr-xr-x 1 root root 4.0M Aug 29 21:33 vrf
4.3M -rwxr-xr-x 1 root root 4.3M Aug 29 21:33 vlan
3.7M -rwxr-xr-x 1 root root 3.7M Aug 29 21:33 tuning
4.3M -rwxr-xr-x 1 root root 4.3M Aug 29 21:33 tap
3.1M -rwxr-xr-x 1 root root 3.1M Aug 29 21:33 static
4.0K -rw-r--r-- 1 root root 2.3K Aug 29 21:33 README.md
4.4M -rwxr-xr-x 1 root root 4.4M Aug 29 21:33 ptp
4.1M -rwxr-xr-x 1 root root 4.1M Aug 29 21:33 portmap
3.6M -rwxr-xr-x 1 root root 3.6M Aug 29 21:33 loopback
 12K -rw-r--r-- 1 root root  12K Aug 29 21:33 LICENSE
4.3M -rwxr-xr-x 1 root root 4.3M Aug 29 21:33 ipvlan
3.6M -rwxr-xr-x 1 root root 3.6M Aug 29 21:33 host-local
4.2M -rwxr-xr-x 1 root root 4.2M Aug 29 21:33 host-device
4.8M -rwxr-xr-x 1 root root 4.8M Aug 29 21:33 firewall
4.3M -rwxr-xr-x 1 root root 4.3M Aug 29 21:33 dummy
 11M -rwxr-xr-x 1 root root  11M Aug 29 21:33 dhcp
4.6M -rwxr-xr-x 1 root root 4.6M Aug 29 21:33 bridge
4.1M -rwxr-xr-x 1 root root 4.1M Aug 29 21:33 bandwidth
3.8M -rwxr-xr-x 1 root root 3.8M Aug 29 21:33 sbr
4.3M -rwxr-xr-x 1 root root 4.3M Aug 29 21:33 macvlan
2.4M -rwxr-xr-x 1 root root 2.4M Jan 24 04:56 flannel
```
</details>

<details><summary>Q3: What is the CNI plugin configured to be used on this kubernetes cluster?</summary>

```bash
/etc/cni/net.d ➜  cat 10-flannel.conflist 
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

The answer is `Flannel` as the `isDefaultGateway` is set to `true`, whereas the `portmap` is for the port mappings.

</details>

------

```bash
$ kubectl get all -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
pod/coredns-77d6fd4654-2n8r9               1/1     Running   0          37m
pod/coredns-77d6fd4654-wwng5               1/1     Running   0          37m
pod/etcd-controlplane                      1/1     Running   0          37m
pod/kube-apiserver-controlplane            1/1     Running   0          37m
pod/kube-controller-manager-controlplane   1/1     Running   0          37m
pod/kube-proxy-qv6mc                       1/1     Running   0          37m
pod/kube-scheduler-controlplane            1/1     Running   0          37m

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   37m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   37m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           37m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-77d6fd4654   2         2         2       37m


Events:
  Type     Reason                  Age                  From               Message
  ----     ------                  ----                 ----               -------
  Normal   Scheduled               2m40s                default-scheduler  Successfully assigned default/app to controlplane
  Warning  FailedCreatePodSandBox  2m40s                kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "f6a19b30ef75e1457c03ba90c7c232d9979484e6f9e7f8d9245fa1ca2eb8377d": plugin type="weave-net" name="weave" failed (add): unable to allocate IP address: Post "http://127.0.0.1:6784/ip/f6a19b30ef75e1457c03ba90c7c232d9979484e6f9e7f8d9245fa1ca2eb8377d": dial tcp 127.0.0.1:6784: connect: connection refused
  Normal   SandboxChanged          7s (x13 over 2m39s)  kubelet            Pod sandbox changed, it will be killed and re-created.
```
```yaml
$ cat weave-daemonset-k8s.yaml 
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: weave-net
      labels:
        name: weave-net
      namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: weave-net
      labels:
        name: weave-net
    rules:
      - apiGroups:
          - ''
        resources:
          - pods
          - namespaces
          - nodes
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - extensions
        resources:
          - networkpolicies
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - 'networking.k8s.io'
        resources:
          - networkpolicies
        verbs:
          - get
          - list
          - watch
      - apiGroups:
        - ''
        resources:
        - nodes/status
        verbs:
        - patch
        - update
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: weave-net
      labels:
        name: weave-net
    roleRef:
      kind: ClusterRole
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: weave-net
      namespace: kube-system
      labels:
        name: weave-net
    rules:
      - apiGroups:
          - ''
        resources:
          - configmaps
        resourceNames:
          - weave-net
        verbs:
          - get
          - update
      - apiGroups:
          - ''
        resources:
          - configmaps
        verbs:
          - create
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: weave-net
      namespace: kube-system
      labels:
        name: weave-net
    roleRef:
      kind: Role
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: weave-net
      labels:
        name: weave-net
      namespace: kube-system
    spec:
      # Wait 5 seconds to let pod connect before rolling next pod
      selector:
        matchLabels:
          name: weave-net
      minReadySeconds: 5
      template:
        metadata:
          labels:
            name: weave-net
        spec:
          initContainers:
            - name: weave-init
              image: 'weaveworks/weave-kube:2.8.1'
              command:
                - /home/weave/init.sh
              env:
              securityContext:
                privileged: true
              volumeMounts:
                - name: cni-bin
                  mountPath: /host/opt
                - name: cni-bin2
                  mountPath: /host/home
                - name: cni-conf
                  mountPath: /host/etc
                - name: lib-modules
                  mountPath: /lib/modules
                - name: xtables-lock
                  mountPath: /run/xtables.lock
                  readOnly: false
          containers:
            - name: weave
              command:
                - /home/weave/launch.sh
              env:
                - name: IPALLOC_RANGE
                  value: 10.32.1.0/24
                - name: INIT_CONTAINER
                  value: "true"
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'weaveworks/weave-kube:2.8.1'
              readinessProbe:
                httpGet:
                  host: 127.0.0.1
                  path: /status
                  port: 6784
              resources:
                requests:
                  cpu: 50m
              securityContext:
                privileged: true
              volumeMounts:
                - name: weavedb
                  mountPath: /weavedb
                - name: dbus
                  mountPath: /host/var/lib/dbus
                  readOnly: true
                - mountPath: /host/etc/machine-id
                  name: cni-machine-id
                  readOnly: true
                - name: xtables-lock
                  mountPath: /run/xtables.lock
                  readOnly: false
            - name: weave-npc
              env:
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'weaveworks/weave-npc:2.8.1'
#npc-args
              resources:
                requests:
                  cpu: 50m
              securityContext:
                privileged: true
              volumeMounts:
                - name: xtables-lock
                  mountPath: /run/xtables.lock
                  readOnly: false
          hostNetwork: true
          dnsPolicy: ClusterFirstWithHostNet
          hostPID: false
          restartPolicy: Always
          securityContext:
            seLinuxOptions: {}
          serviceAccountName: weave-net
          tolerations:
            - effect: NoSchedule
              operator: Exists
            - effect: NoExecute
              operator: Exists
          volumes:
            - name: weavedb
              hostPath:
                path: /var/lib/weave
            - name: cni-bin
              hostPath:
                path: /opt
            - name: cni-bin2
              hostPath:
                path: /home
            - name: cni-conf
              hostPath:
                path: /etc
            - name: cni-machine-id
              hostPath:
                path: /etc/machine-id
            - name: dbus
              hostPath:
                path: /var/lib/dbus
            - name: lib-modules
              hostPath:
                path: /lib/modules
            - name: xtables-lock
              hostPath:
                path: /run/xtables.lock
                type: FileOrCreate
          priorityClassName: system-node-critical
      updateStrategy:
        type: RollingUpdate
```
```bash
$ kubectl apply -f weave-daemonset-k8s.yaml 
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created


$ kubectl get all -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
pod/coredns-77d6fd4654-2n8r9               1/1     Running   0          40m
pod/coredns-77d6fd4654-wwng5               1/1     Running   0          40m
pod/etcd-controlplane                      1/1     Running   0          40m
pod/kube-apiserver-controlplane            1/1     Running   0          40m
pod/kube-controller-manager-controlplane   1/1     Running   0          40m
pod/kube-proxy-qv6mc                       1/1     Running   0          40m
pod/kube-scheduler-controlplane            1/1     Running   0          40m
pod/weave-net-g7n2b                        2/2     Running   0          35s

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   40m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   40m
daemonset.apps/weave-net    1         1         1       1            1           <none>                   35s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           40m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-77d6fd4654   2         2         2       40m
```

- Official Weaveworks binary packages: https://github.com/weaveworks/weave/releases


-----

### IPAM - IP Address Management

![ipam_weave_cni_1](ipam_weave_cni_1.png)

![ipam_weave_cni_2](ipam_weave_cni_2.png)

Two CNI plugins who'll manages IP address:
- DHCP
- host-local

```bash
$ cat /etc/cni/net.d/net-script.conf

# Weavenet can be set within the IPs: 10.32.0.0./12 -> 10.32.0.1 > 10.47.266.254

$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 2e:b9:8a:5d:e2:7f brd ff:ff:ff:ff:ff:ff
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether fa:67:d9:50:df:5a brd ff:ff:ff:ff:ff:ff
6: vethwe-datapath@vethwe-bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master datapath state UP mode DEFAULT group default 
    link/ether 72:36:03:07:ca:d7 brd ff:ff:ff:ff:ff:ff
7: vethwe-bridge@vethwe-datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP mode DEFAULT group default 
    link/ether 7a:0b:4e:67:1b:d9 brd ff:ff:ff:ff:ff:ff
8: vxlan-6784: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65535 qdisc noqueue master datapath state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether d2:46:e8:5b:5c:87 brd ff:ff:ff:ff:ff:ff
10: vethweplfc6ca6f@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP mode DEFAULT group default 
    link/ether 36:d8:eb:3a:df:8c brd ff:ff:ff:ff:ff:ff link-netns cni-e0d7d33a-e519-f66f-1a70-706cb741016e
12: vethweplbedcb6e@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP mode DEFAULT group default 
    link/ether 4e:d2:de:29:f3:2e brd ff:ff:ff:ff:ff:ff link-netns cni-422efb75-bfbd-772e-3b59-f4cb7ec6ef31
6205: eth0@if6206: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:c0:0a:13:06 brd ff:ff:ff:ff:ff:ff link-netnsid 0
6207: eth1@if6208: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:ac:19:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 1

$ /etc/cni/net.d ➜  ip link show type bridge
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether fa:67:d9:50:df:5a brd ff:ff:ff:ff:ff:ff
```
-----

### Service Networking

Basically, the services that are created and provided an IP address but, there is no pod, no network namespace, or container created, it is just an object created.

Please note, the `clusterIP` and `nodePort` services are cluster-wide objects where, the services are accessible from all the nodes. The IP addressed associated to those services are basically done through a Network Address Translation table.

**In the below image, if you note, the `service IP:Port` is the service IP and port whereas, it is forwarded to the `pod IP:Port`:**

![service_networking_nat_config_1](service_networking_nat_config_1.png)

![service_networking_kube_proxy_iptables_2](service_networking_kube_proxy_iptables_2.png)

```bash
# you can find the kubeproxy logs updating the service details 
$ cat /var/log/kube-proxy.log
```

------

##### Questions & Commands Used:

<details><summary>Q1: What network range are the nodes in the cluster part of?</summary>

```bash
$ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane   27m   v1.31.0   192.11.78.6   <none>        Ubuntu 22.04.4 LTS   5.4.0-1106-gcp   containerd://1.6.26
node01         Ready    <none>          26m   v1.31.0   192.11.78.9   <none>        Ubuntu 22.04.4 LTS   5.4.0-1106-gcp   containerd://1.6.26
```

Seems like the nodes are part of the network range: `192.11.78.0/24`

</details>

<details><summary>Q2: What is the range of IP addresses configured for PODs on this cluster?</summary>

To understand this, let create a new pod and check the IP address:
```bash
$ kubectl run nginx --image=nginx
pod/nginx created

$ kubectl get pods -o wide
NAME    READY   STATUS              RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
nginx   0/1     ContainerCreating   0          5s    <none>   node01   <none>           <none>

$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          8s    10.244.192.1   node01   <none>           <none>
```

Seems like the pods are part of the network range: `10.244.192.0/24` or `10.244.0.0/16`

</details>


<details><summary>Q3: What is the IP Range configured for the services within the cluster?</summary>

To understand this, let's check the kube-apiserver where it passes: `--service-cluster-ip-range=10.96.0.0/12`

```bash
$ ps -aux | grep -i "kube-apiserver"
root        3648  0.0  0.1 1525984 284272 ?      Ssl  12:51   1:24 kube-apiserver --advertise-address=192.10.197.12 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root       11079  0.0  0.0   6936  2232 pts/2    S+   13:35   0:00 grep --color=auto -i kube-apiserver
```

Seems like the services are part of the network range: `10.96.0.0/12`

</details>

<details><summary>Q4: How many kube-proxy pods are deployed in this cluster?</summary> 

```bash
$ kubectl get pods -A -o wide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS      AGE   IP            NODE           NOMINATED NODE   READINESS GATES
kube-system   coredns-77d6fd4654-kc28n               1/1     Running   0             29m   10.244.0.3    controlplane   <none>           <none>
kube-system   coredns-77d6fd4654-n6bvc               1/1     Running   0             29m   10.244.0.2    controlplane   <none>           <none>
kube-system   etcd-controlplane                      1/1     Running   0             29m   192.11.78.6   controlplane   <none>           <none>
kube-system   kube-apiserver-controlplane            1/1     Running   0             29m   192.11.78.6   controlplane   <none>           <none>
kube-system   kube-controller-manager-controlplane   1/1     Running   0             29m   192.11.78.6   controlplane   <none>           <none>
kube-system   kube-proxy-5knwl                       1/1     Running   0             29m   192.11.78.9   node01         <none>           <none>
kube-system   kube-proxy-bxmt6                       1/1     Running   0             29m   192.11.78.6   controlplane   <none>           <none>
kube-system   kube-scheduler-controlplane            1/1     Running   0             29m   192.11.78.6   controlplane   <none>           <none>
kube-system   weave-net-cgvxb                        2/2     Running   0             29m   192.11.78.9   node01         <none>           <none>
kube-system   weave-net-dbgkx                        2/2     Running   1 (29m ago)   29m   192.11.78.6   controlplane   <none>           <none>
```

There are two `kube-proxy pods` - `kube-proxy-5knwl and kube-proxy-bxmt6`

</details>

<details><summary>Q5: What `type` of proxy is the kube-proxy configured to use?</summary> 

```bash
kubectl logs pod/kube-proxy-5knwl -n kube-system
I0124 12:52:20.968536       1 server_linux.go:66] "Using iptables proxy"
I0124 12:52:21.140070       1 server.go:677] "Successfully retrieved node IP(s)" IPs=["192.10.197.3"]
I0124 12:52:21.160761       1 conntrack.go:60] "Setting nf_conntrack_max" nfConntrackMax=1179648
I0124 12:52:21.162817       1 conntrack.go:121] "Set sysctl" entry="net/netfilter/nf_conntrack_tcp_timeout_established" value=86400
E0124 12:52:21.164320       1 server.go:234] "Kube-proxy configuration may be incomplete or incorrect" err="nodePortAddresses is unset; NodePort connections will be accepted on all local IPs. Consider using `--nodeport-addresses primary`"
I0124 12:52:21.186680       1 server.go:243] "kube-proxy running in dual-stack mode" primary ipFamily="IPv4"
I0124 12:52:21.186735       1 server_linux.go:169] "Using iptables Proxier"
I0124 12:52:21.189087       1 proxier.go:255] "Setting route_localnet=1 to allow node-ports on localhost; to change this either disable iptables.localhostNodePorts (--iptables-localhost-nodeports) or set nodePortAddresses (--nodeport-addresses) to filter loopback addresses" ipFamily="IPv4"
I0124 12:52:21.207947       1 server.go:483] "Version info" version="v1.31.0"
I0124 12:52:21.207991       1 server.go:485] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
I0124 12:52:21.211185       1 config.go:197] "Starting service config controller"
I0124 12:52:21.211211       1 config.go:326] "Starting node config controller"
I0124 12:52:21.211227       1 shared_informer.go:313] Waiting for caches to sync for node config
I0124 12:52:21.211229       1 shared_informer.go:313] Waiting for caches to sync for service config
I0124 12:52:21.211325       1 config.go:104] "Starting endpoint slice config controller"
I0124 12:52:21.211340       1 shared_informer.go:313] Waiting for caches to sync for endpoint slice config
I0124 12:52:21.311388       1 shared_informer.go:320] Caches are synced for node config
I0124 12:52:21.311480       1 shared_informer.go:320] Caches are synced for endpoint slice config
I0124 12:52:21.311574       1 shared_informer.go:320] Caches are synced for service config
```

From this logline: `Using iptables proxy` -> We could see that the kube-proxy is using the `iptables proxy`.

</details>

<details><summary>Q5: How does this Kubernetes cluster ensure that a kube-proxy pod runs on all nodes in the cluster?</summary> 

```bash
$ kubectl get pod/kube-proxy-5knwl -n kube-system -o yaml | grep -i "kind"
kind: Pod
    kind: DaemonSet
```

Yes, it is deployed as `DaemonSets` across all the nodes.
</details>

-----

### Cluster DNS

Let's see:

- What names are assigned to what objects
- Service DNS records
- POD DNS records


![creating_service_and_reaching_them_how_DNS_stores_it_1](creating_service_and_reaching_them_how_DNS_stores_it_1.png)

It is the same way, it stores the details of the pods as well. Where, you can reach the service via: `https://webservice.apps.svc.cluster.local` but, for pods(if there is no services in front), we can reach to `https://testpod.default.pod.cluster.local`.

![creating_service_and_reaching_them_how_DNS_stores_it_2](creating_service_and_reaching_them_how_DNS_stores_it_2.png)

>[!Important]
>Please note, for Pods, the DNS in K8s,  interchanges the IP address (dots) i.e. 10.244.2.5 to 10-244-2-5 replaced with (dashes) as `hostnames` in the DNS records.

----

### CoreDNS in Kubernetes

#### How Kubernetes Implements DNS in the cluster

Prior to the K8s v1.21, it was using the `kube-DNS`, but post that, to the latest versions, it started using the `coreDNS`.

So, basically, all the configurations and plugins should be in the `/etc/coredns/Corefile`

![coreDNS_corefile_plugin_for_kubernetes](coreDNS_corefile_plugin_for_kubernetes.png)

Please note, once the CoreDNS is deployed, it is deployed as `ReplicaSets` so it can have redundancy. And, the `coreDNS pods` are available to serve the pods.

Whenever a pod is deployed, the `kubelet` also includes a line in the `/etc/resolv.conf` as the `nameserver <coreDNS-internal-ip>`. This is because the `kubelet` is responsible to deploy the new pods and it'll take care of those.

![coreDNS_for_kubernetes_for_pods](coreDNS_for_kubernetes_for_pods.png)

**In Simple scenario** - If there are two pods in the `default` namespace, we can add both the pods IP address in the `/etc/hosts` with the `hostname` so that when we type `ping podB`, it can reach. But, when the cluster is scaled, it is NOT possible to do the same updation in each pods everytime, which is why, we have the `nameserver` in place. And, the coreDNS will take the responsibility of resolving the domain names for the pods when reached.

##### Commands Used & Questions

<details><summary>
</summary>
</details>

