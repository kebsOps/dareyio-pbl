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

**INSTALL CLIENT TOOLS BEFORE BOOTSTRAPPING THE CLUSTER**

You will a few client tools installed and configurations made on your client workstation which include the following:

**- awscli**  is a unified tool to manage your AWS services

**- kubectl**  this command line utility will be your main control tool to manage your K8s cluster. 

**- cfssl**  an open source toolkit for everything TLS/SSL from Cloudflare

**- cfssljson**  a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.

**Install and configure AWS CLI**

Configure AWS CLI to access all AWS services used, for this you need to have a user with programmatic access keys configured in AWS Identity and Access Management (IAM):

On your local workstation download and install the latest version of AWS CLI

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/b7ffbc0e-a622-4833-8f03-6289e2fe1822)

<img width="900" alt="Screenshot 2023-10-12 at 14 46 24" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/1565d366-fd36-425a-b345-a565d6e65f66">

**Intalling Kubectl**

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/1ac24ec9-15ab-4e90-bd92-17f0e29770e9)

