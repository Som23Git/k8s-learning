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


