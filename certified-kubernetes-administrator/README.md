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