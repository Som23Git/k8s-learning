This is a K8s Learning Repo for self understanding


01/12/2024

### Important Components in K8s

- API Server -- This is the server that handles all the CLI commands i.e. Kubectl commands
- ETCD -- This is a key-value store
- Kubelet -- Agent that runs on the Worker node
- Container Runtime -- This is a software that runs on Worker node
- Scheduler -- This schedules the cron job
- Controller -- Monitors the cluster state, it got additional controllers like Node Controllers, Deployment Controllers, and the Endpoint Controllers.

Where,

```
# In the master node, only 4 components will be there to control and store information

API Server, ETCD, Scheduler, Controller

# In the Worker node, there will be only 2 components

Kubelet, Container Runtime
```

### Detailed information on the Key components

---

### **1. Master Node**
The Master Node is responsible for managing the Kubernetes cluster. It handles scheduling, maintaining the desired state, and orchestrating workloads. The components of the Master Node include:

#### **a. API Server**
- Acts as the front-end for the Kubernetes control plane.
- Exposes the Kubernetes API, allowing communication between components and with external users (via `kubectl`, REST clients, etc.).

#### **b. Controller Manager**
- Runs controllers that monitor the cluster's state and make changes to maintain the desired state.
- Includes controllers such as:
  - Node Controller: Manages node availability.
  - Deployment Controller: Handles the desired state of deployments.
  - Endpoints Controller: Manages endpoint objects for services.

#### **c. Scheduler**
- Assigns pods to nodes based on resource requirements and constraints.
- Ensures optimal distribution of workloads.

#### **d. etcd**
- A distributed key-value store used for storing all cluster data, including configuration, state, and metadata.
- Ensures consistency across the cluster.

---

### **2. Worker Nodes**
Worker Nodes host the application workloads. They run and manage pods, which contain containerized applications. Key components of Worker Nodes include:

#### **a. Kubelet**
- An agent running on each node that ensures containers in pods are running.
- Communicates with the API Server to receive instructions and report the node's status.

#### **b. Kube-Proxy**
(Something that we did not discuss as a key component)
- A network proxy that manages communication and load balancing for services.
- Routes traffic to the appropriate pods across nodes.

#### **c. Container Runtime**
- Software responsible for running containers (e.g., Docker, containerd, CRI-O).
- Interfaces with Kubernetes via the Container Runtime Interface (CRI).

---

### **3. Additional Key Components**

#### **a. Pods**
- The smallest deployable unit in Kubernetes.
- Encapsulates one or more containers, storage, network, and configuration.

#### **b. Services**
- Provide stable networking for accessing a set of pods.
- Enable load balancing and expose pods inside or outside the cluster.

#### **c. ConfigMaps and Secrets**
- ConfigMaps: Store configuration data in a key-value format, allowing decoupling from application logic.
- Secrets: Securely store sensitive data like passwords or API keys.

#### **d. Persistent Volumes (PV) and Persistent Volume Claims (PVC)**
- PV: Provides storage abstraction.
- PVC: Requests specific storage resources from PVs.

#### **e. Ingress**
- Manages external HTTP/HTTPS access to services within the cluster.
- Provides features like URL-based routing and SSL termination.

#### **f. Cluster Networking**
- Enables communication between pods, nodes, and external systems.
- Includes CNI (Container Network Interface) plugins for networking.

---

### **4. Add-ons**
Enhance Kubernetes functionality and are often deployed as pods or services:

#### **a. Dashboard**
- A web-based UI for managing and troubleshooting the cluster.

#### **b. Monitoring**
- Tools like Prometheus and Grafana for tracking resource usage and cluster health.

#### **c. Logging**
- Aggregates logs from pods and nodes for centralized monitoring (e.g., Elastic Stack, Fluentd).

#### **d. DNS**
- Provides cluster-internal DNS resolution for services and pods.

---

### **High-Level Interaction**
1. Users interact with the **API Server** using `kubectl` or other clients.
2. The **Scheduler** assigns workloads to appropriate nodes.
3. **Controller Manager** ensures resources and workloads meet the desired state.
4. The **Worker Nodes** execute workloads, managed by the **Kubelet** and **Kube-Proxy**.

This architecture ensures Kubernetes can dynamically orchestrate and scale containerized applications efficiently.

### K8s Architecture
```
kubectl run hello-minikube
```

```
kubectl cluster-info
```

```
kubectl get nodes
```

### Container Run Time

- Docker (running on ELXI container orchestration)
- RKT
- CRI-O

### Docker Vs ContainerD

What is important? Based on the learnings, it seems like the using K8s via Docker is discontinued from K8s v1.24. However, to our understanding, the underlying container runtime for Docker was also ContainerD but, we do not directly interact with it.

So, what K8s community did, is to create two tools that can save time and be efficient to communicate with the K8s architecture:
[1] `nerdctl` - Can be used widely and supports only ContainerD runtime.
[2] `crictl` - Container Runtime Interface(cri) can be used too and supports most of the runtime including RKT, CRI-O, and ContainerD as well.

### K8s Concepts

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

### YAML and how to construct it?

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

### Pods with YAML

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
kubectl create -f replicaset-definition.yaml

kubectl get replicaset

kubectl delete <replicaset_name>     ---- This deletes all the pods underlying the replicasets

kubectl replace -f replicaset-definition.yml

kubectl scale --replicas=6 -f replicaset.yml
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

```
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

```
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

```
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

```

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

Then, to delete all the pod in one go:

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

### Deployments

```
$ kubectl create -f deployment-definition.yaml
deployment.apps/app-deployment created
```
```
$ kubctl get all 

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

```
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

```
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