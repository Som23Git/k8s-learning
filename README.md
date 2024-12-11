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

#### Deployment Updates, Rollout, and Versioning

Commands you to run Rollout strategy: 
[1] Rolling update 
[2] Recreate

```
kubectl create -f deployment-definition.yml
kubectl get deployments
kubectl apply -f deployment-definition.yml
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
kubectl create -f deployment-definition.yml --record=true
```

```
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

### Networking in K8s

```
$ kubectl get all -o wide

NAME                                        READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/app-deployment-nginx-69976db885-8qzx9   1/1     Running   0             12m   10.244.0.45   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-fsp52   1/1     Running   0             12m   10.244.0.44   minikube   <none>           <none>
pod/app-deployment-nginx-69976db885-jpc65   1/1     Running   0             12m   10.244.0.43   minikube   <none>           <none>
pod/hello-node-7b4b746b66-9cr8x             1/1     Running   3 (13h ago)   45d   10.244.0.23   minikube   <none>           <none>
```
As you see here: the pods have got IPs under the same node that's why we could see this:

```
IP
10.244.0.45
10.244.0.44
10.244.0.43
10.244.0.23
```

And, this IP addresses are given to the pods in the node below:

```
$ kubectl get nodes -o wide

NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   45d   v1.30.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   5.15.49-linuxkit   docker://26.1.1
```

So, the node IP address is `192.168.49.2` and the pods are given the IP addresses above: `10.244.0.45, 10.244.0.44, 10.244.0.43, 10.244.0.23`

Here's how the ASCII Art i.e. K8s architecture for networking or IPs is used:

```
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
| +-----------------------------------------------------+   |
|                                                           |
+-----------------------------------------------------------+

+-----------------------------------------------------------+
|                   Standard K8s Cluster                    |
|-----------------------------------------------------------|
| +-----------------------------------------------------+   |
| |                     Node 1                          |   |
| |-----------------------------------------------------|   |
| | Node IP: 192.168.1.1                                |   |
| | Pod CIDR: 10.244.1.0/24                             |   |
| | +--------------------+   +-----------------------+  |   |
| | | Pod A              |   | Pod B                 |  |   |
| | | IP: 10.244.1.2     |   | IP: 10.244.1.3        |  |   |
| | +--------------------+   +-----------------------+  |   |
| +-----------------------------------------------------+   |
|                                                           |
| +-----------------------------------------------------+   |
| |                     Node 2                          |   |
| |-----------------------------------------------------|   |
| | Node IP: 192.168.1.2                                |   |
| | Pod CIDR: 10.244.2.0/24                             |   |
| | +--------------------+   +-----------------------+  |   |
| | | Pod C              |   | Pod D                 |  |   |
| | | IP: 10.244.2.2     |   | IP: 10.244.2.3        |  |   |
| | +--------------------+   +-----------------------+  |   |
| +-----------------------------------------------------+   |
|                                                           |
| Service Cluster IP Range: 10.96.0.0/12                    |
|                                                           |
+-----------------------------------------------------------+

```


### Services in Kubernetes

[1] Services enable connectivity with the internal components and external components.
[2] Services can work with the applications internally.

There are three different types of Services:

- NodePort
- ClusterIP
- LoadBalancer

#### NodePort Service

```
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

####$ Example:

- Before creating the service:


```
$ kubectl get all -o wide   

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/hello-node   LoadBalancer   10.103.22.214   <pending>     8080:31963/TCP   46d   k8s-app=hello-node
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          46d   <none>
```

- Post creating the service using the below definition file:

```
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
      nodePort: 30007             # port that is in the node that gets exposed to the outside world or internet
  selector:
    app: prod-app
```

```
$ kubectl create -f service-definition.yaml 
service/nginx-service created
```

```
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

```
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
```
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

```
kubectl describe service <service_name>

OR

kubectl describe svc <service_name>
```

> [!NOTE]
> Make use of the imperative commands i.e. the shortcut commands used in K8s from the [kubernetes official documentation](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/).

### Microservices Architecture

Let's create a voting app and result app in K8s considering a Microservice Architecture:

Please refer to the files in the directory [voting-app](./voting-app)

Once you execute, `kubectl create -f .`, it will create all the pods and the services associated with the pod for the voting app.

The output looks like this:

```
$ kubectl get all -o wide    

NAME                              READY   STATUS    RESTARTS        AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod/postgres-pod                  1/1     Running   0               16m   10.244.0.53   minikube   <none>           <none>
pod/redis                         1/1     Running   0               34m   10.244.0.47   minikube   <none>           <none>
pod/result-app-pod                1/1     Running   3 (16m ago)     34m   10.244.0.48   minikube   <none>           <none>
pod/voting-app-pod                1/1     Running   0               34m   10.244.0.49   minikube   <none>           <none>
pod/worker                        1/1     Running   6 (8m54s ago)   12m   10.244.0.54   minikube   <none>           <none>

NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE     SELECTOR
service/db               ClusterIP      10.102.64.116    <none>        5432/TCP         23m     pod=db-pod
service/kubernetes       ClusterIP      10.96.0.1        <none>        443/TCP          47d     <none>
service/nginx-service    NodePort       10.102.73.198    <none>        80:30007/TCP     24h     app=prod-app
service/redis            ClusterIP      10.100.115.238   <none>        6379/TCP         6m50s   pod=redis
service/result-service   NodePort       10.104.9.71      <none>        80:30081/TCP     6m50s   pod=result-app-pod
service/voting-service   NodePort       10.105.162.209   <none>        80:30080/TCP     111s    pod=voting-app-pod

```

And, once you make sure all the services and pods are `UP` and `Running`, you can get the `service URL` from the `minikube` command:

For voting-service:
```
minikube service voting-service --url
http://127.0.0.1:49609

```

And, for result-service:

```
minikube service result-service --url
http://127.0.0.1:49639

```
Now, you should be able to access the application and could notice the change in the number of votes.

> ![Important]
> For more details on the code and the repository: we can check this [kodekloud's repo](https://github.com/kodekloudhub/example-voting-app/tree/master/k8s-specifications)

### Kubernetes on Cloud

IN GKE or AWS or AKS:
We need to change the `NodePort` type of the `Service` to `LoadBalancer` type in voting-app service and result-ap service.

```
apiVersion: v1
kind: Service
metadata:
  name: voting-service
  labels:
    name: voting-service
    app: demo-voting-app
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    name: voting-app-pod
    app: demo-voting-app
```

And,

```
apiVersion: v1
kind: Service
metadata:
  name: result-service
  labels:
    name: result-service
    app: demo-voting-app
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    name: result-app-pod
    app: demo-voting-app
```

Once you deploy all the pods and services in the GKE or other cloud providers, it will assign the `external IP` for the LoadBalancer services so that we can reach the application from outside.

```
$ kubectl get all -o wide

NAME                                     READY   STATUS    RESTARTS   AGE   IP           NODE                                             NOMINATED NODE   READINESS GATES
pod/postgres-deploy-776f4bdfcb-mlg22     1/1     Running   0          87s   10.120.1.3   gke-som-gke-cluster-default-pool-98c25668-kkvj   <none>           <none>
pod/redis-deploy-54b67b7b4-r7z2t         1/1     Running   0          86s   10.120.7.3   gke-som-gke-cluster-default-pool-5a6cf592-b2g0   <none>           <none>
pod/result-app-deploy-7cc95854b6-jqd7b   1/1     Running   0          85s   10.120.3.4   gke-som-gke-cluster-default-pool-d134ac34-0x9r   <none>           <none>
pod/voting-app-deploy-96f58547f-r5lg8    1/1     Running   0          84s   10.120.8.4   gke-som-gke-cluster-default-pool-5a6cf592-jvj6   <none>           <none>
pod/worker-app-deploy-54575bd48c-kgk2f   1/1     Running   0          84s   10.120.0.4   gke-som-gke-cluster-default-pool-98c25668-h2pc   <none>           <none>

NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE     SELECTOR
service/db               ClusterIP      34.118.226.245   <none>          5432/TCP       86s     app=demo-voting-app,name=postgres-pod
service/kubernetes       ClusterIP      34.118.224.1     <none>          443/TCP        8m16s   <none>
service/redis            ClusterIP      34.118.228.197   <none>          6379/TCP       85s     app=demo-voting-app,name=redis-pod
service/result-service   LoadBalancer   34.118.235.13    34.72.60.201    80:31723/TCP   85s     app=demo-voting-app,name=result-app-pod
service/voting-service   LoadBalancer   34.118.228.248   34.123.71.190   80:32610/TCP   84s     app=demo-voting-app,name=voting-app-pod

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                 SELECTOR
deployment.apps/postgres-deploy     1/1     1            1           88s   postgres     postgres                               app=demo-voting-app,name=postgres-pod
deployment.apps/redis-deploy        1/1     1            1           87s   redis        redis                                  app=demo-voting-app,name=redis-pod
deployment.apps/result-app-deploy   1/1     1            1           86s   result-app   kodekloud/examplevotingapp_result:v1   app=demo-voting-app,name=result-app-pod
deployment.apps/voting-app-deploy   1/1     1            1           86s   voting-app   kodekloud/examplevotingapp_vote:v1     app=demo-voting-app,name=voting-app-pod
deployment.apps/worker-app-deploy   1/1     1            1           85s   worker-app   kodekloud/examplevotingapp_worker:v1   app=demo-voting-app,name=worker-app-pod

NAME                                           DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                 SELECTOR
replicaset.apps/postgres-deploy-776f4bdfcb     1         1         1       88s   postgres     postgres                               app=demo-voting-app,name=postgres-pod,pod-template-hash=776f4bdfcb
replicaset.apps/redis-deploy-54b67b7b4         1         1         1       87s   redis        redis                                  app=demo-voting-app,name=redis-pod,pod-template-hash=54b67b7b4
replicaset.apps/result-app-deploy-7cc95854b6   1         1         1       86s   result-app   kodekloud/examplevotingapp_result:v1   app=demo-voting-app,name=result-app-pod,pod-template-hash=7cc95854b6
replicaset.apps/voting-app-deploy-96f58547f    1         1         1       85s   voting-app   kodekloud/examplevotingapp_vote:v1     app=demo-voting-app,name=voting-app-pod,pod-template-hash=96f58547f
replicaset.apps/worker-app-deploy-54575bd48c   1         1         1       85s   worker-app   kodekloud/examplevotingapp_worker:v1   app=demo-voting-app,name=worker-app-pod,pod-template-hash=54575bd48c
```

### K8s Setup with Kubeadm

Kubeadm - called as KubeAdmin which will take care of complete installation and management of the production-grade k8s cluster. Kubeadm is the starting point of how you can manage the cluster's TLS certs, pod network, connection between the Master and the Worker Nodes.

It keeps things simple and straightforward. This is the tool most of the GKE, AKS, AWS EKS are using to deploy managed K8s solution in the cloud infrastructure.

We can make use of the `Vagrant` tool to install a self-managed K8s cluster with 1 master node and 2 worker nodes.


### K8s Setup with Minikube

Minikube is a all-in-one K8s box where, it deploys a node(master node/worker node) that got all the K8s components i.e. API server, etcd(key-value store), controllers i.e. node controller and replica controller, kubelet, scheduler, and the container runtime i.e. containerD in my case. You may use the virual box as the driver and as a container runtime too. Please check the below links:

Install and set up the kubectl tool: https://kubernetes.io/docs/tasks/tools/

Install Minikube: https://minikube.sigs.k8s.io/docs/start/

Install VirtualBox: https://www.virtualbox.org/wiki/Downloads

https://www.virtualbox.org/wiki/Linux_Downloads

Minikube Tutorial: https://kubernetes.io/docs/tutorials/hello-minikube/

If the minikube installation has been done on the macOS, then to access the URL on the local browser, we need to do a few steps to get the service URL to work. Those steps are covered on this documentation page: https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-service-with-tunnel