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



