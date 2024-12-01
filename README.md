This is a K8s Learning Repo for self understanding


01/12/2024

### Important Components in K8s

- API Server -- This is the server that handles all the CLI commands i.e. Kubectl commands
- ETCD -- This is a key-value store
- Kubelet -- Agent that runs on the Worker node
- Container Runtime -- This is a software that runs on Worker node
- Scheduler -- This schedules the cron job
- Controller

Where,

```
# In the master node, only 4 components will be there to control and store information

API Server, ETCD, Scheduler, Controller

# In the Worker node, there will be only 2 components

Kubelet, Container Runtime
```

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


