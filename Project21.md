## Orchestrating Containers across Multiple Virtual Servers with Kubernetes

### Kubernetes Architecture
Kubernetes is a not a single package application that you can install with one command, it is comprised of several components, 
some of them can be deployed as services, some can be also deployed as separate containers.

![K8s_architecture](https://github.com/kebsOps/dareyio-pbl/assets/10085348/74e6a14b-7f43-4df9-adcd-67873584029f)


**To successfully implement "K8s From-Ground-Up", the following and even more will be done by you as a K8s administrator:**

1. Install and configure master (also known as control plane) components and worker nodes (or just nodes).
1. Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
    - In transit encryption means encrypting communications over the network using HTTPS
    - At rest encryption means encrypting the data stored on a disk
1. Plan the capacity for the backend data store etcd
1. Configure network plugins for the containers to communicate
1. Manage periodical upgrade of the cluster
1. Configure observability and auditing
