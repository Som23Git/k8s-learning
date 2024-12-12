11th Dec - 
# Certified Kubernetes Adminitrator

https://www.cncf.io/certification/cka

Mode of delivery: Online
Online Proctor will be available

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

### Core Concepts

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

> ![NOTE]
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

### Kube-API Server

General responsibilities of the `kube-api` server:

- Authenticate User
- Validate request
- Retrieve data from ETCD datastore
- Update ETCD
- Scheduler
- Kubelet

> ![NOTE]
> When using the `kubeadm` tool, it deploys kube-api server on our behalf and it does the heavy-lifting but, to understand how it all configured, you would need to download the binary of the `kube-apiserver` and add the configuration flags/parameters to connect to the cluster accordingly.

#### Commands that you can use to find the kube-apiserver

```
# When using `kubeadm` it deploys the kube-apiserver as a pod

$ kubectl get pods -n kube-system

To get the configurations of the kube-apiserver, we can check:

$ cat /etc/kubernetes/manifests/kube-apiserver.yaml

# When deploying it from scratch using binary, where it is a non-kubeadm setup:

$ cat /etc/systemd/system/kube-apiserver.service

# Or, you can use the process command to check the parameters passed to run the kube-apiserver

$ ps -aux | grep -e "kube-apiserver"
```

### Kube Controller Manager


