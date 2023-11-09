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

### Install and configure AWS CLI

Configure AWS CLI to access all AWS services used, for this you need to have a user with programmatic access keys configured in AWS Identity and Access Management (IAM):

On your local workstation download and install the latest version of AWS CLI

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/b7ffbc0e-a622-4833-8f03-6289e2fe1822)

<img width="900" alt="Screenshot 2023-10-12 at 14 46 24" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/1565d366-fd36-425a-b345-a565d6e65f66">


## Install Kubectl

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/1ac24ec9-15ab-4e90-bd92-17f0e29770e9)


## Install CFSSL and CFSSLJSON

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/617cc40a-f14c-4dcd-a3df-b1045a22c3b9)


## AWS CLOUD RESOURCES FOR KUBERNETES CLUSTER

As we already know, we need some machines to run the control plane and the worker nodes. In this section, we will provision EC2 Instances required to run the K8s cluster. 
<
Terraform IaC tool was my preferred tool for this project

The following resources were setup:

- VPC
- DHCP 
- Internet Gateway
- Route Tables
- Security Groups for Cluster
- Network Load Balancer and Listener
- Target Groups
- 6 EC2 Instances 3 Master 3 Worker

 [Link to terraform code ](https://github.com/kebsOps/k8s-cluster-from-ground-up)


## PREPARE THE SELF-SIGNED CERTIFICATE AUTHORITY AND GENERATE TLS CERTIFICATES


### The following components running on the Master node will require TLS certificates.
- kube-controller-manager
- kube-scheduler
- etcd
- kube-apiserver
  
### The following components running on the Worker nodes will require TLS certificates.
- kubelet
- kube-proxy

We will provision a **PKI** Infrastructure using cfssl which will have a Certificate Authority. The CA will then generate certificates for all the individual components.

### Self-Signed Root Certificate Authority (CA)
- We provision a CA that will be used to sign additional TLS certificates.

  Create a directory and cd into it:

``mkdir ca-authority && cd ca-authority``

Generate the CA configuration file, Root Certificate, and Private key:

```
 {

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "DAREY.IO DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

The file defines the following:

```
CN – Common name for the authority

algo – the algorithm used for the certificates

size – algorithm size in bits

C – Country

L – Locality (city)

ST – State or province

O – Organization

OU – Organizational Unit
```

**Output:**

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/ac5a065f-eff3-4066-afaf-e4767e4dbbb0)


List the directory to see the created files

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/6ebabfea-3e7f-4054-9b94-a710e63978ae)

The 3 important files here are:

**ca.pem** – The Root Certificate
>
**ca-key.pem** – The Private Key
>
**ca.csr** – The Certificate Signing Request


### Generating TLS Certificates For Client and Server

Using the CA, we provision TLS certs for the following
- kube-controller-manager
- kube-scheduler
- etcd
- kubelet
- kube-proxy
- Kubernetes Admin User
