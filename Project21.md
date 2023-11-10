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

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bd12e010-e58b-4ed1-8380-ac7b61c9e07e)


### Distributing the Client and Server Certificates

Now it is time to start sending all the client and server certificates to their respective instances.

### Sending to Worker Instance Servers

```
for i in 0 1 2; do
  instance="k8s-cluster-worker-${i}"
  echo "Retrieving public IP for ${instance}..."
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text)

  if [ -z "$external_ip" ]; then
    echo "No IP address found for ${instance}"
  else
    echo "Found IP ${external_ip} for ${instance}. Starting SCP..."
    scp  -i ../ssh/k8s-cluster-from-ground-up.id_rsa \
      ca.pem k8s-cluster-from-ground-up-worker-${i}-key.pem k8s-cluster-from-ground-up-worker-${i}.pem ubuntu@${external_ip}:~/
    if [ $? -eq 0 ]; then
      echo "Files copied to ${instance} successfully."
    else
      echo "Error copying files to ${instance}."
    fi
  fi
done
![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/3d20d376-bb39-4474-8c4a-8999b197e0f2)

```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/6f7bfce2-61c2-464f-9f50-b236a2d898f5)


### Sending to Master Instance Servers

```
for i in 0 1 2; do
  instance="k8s-cluster-master-${i}"
  echo "Retrieving public IP for ${instance}..."
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text)

  if [ -z "$external_ip" ]; then
    echo "No IP address found for ${instance}"
  else
    echo "Found IP ${external_ip} for ${instance}. Starting SCP..."
    scp  -i ../ssh/k8s-cluster-from-ground-up.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem master-kubernetes.pem master-kubernetes-key.pem ubuntu@${external_ip}:~/
    if [ $? -eq 0 ]; then
      echo "Files copied to ${instance} successfully."
    else
      echo "Error copying files to ${instance}."
    fi
  fi
done


```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/8588314a-e5bc-45cb-9bbe-970cd4334048)


### Use kubectl to Generate Kubernetes Configuration Files for Authentication

First, let us create a few environment variables for reuse by multiple commands

``KUBERNETES_API_SERVER_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')``


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/af327401-7ebb-4bcf-85f4-36f30f85ff04)

### Send kubeconfig to worker nodes

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/7d7ce564-434a-4427-89a9-c1b511149880)


### Send kubeconfig to master nodes

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/ede6e41e-361e-4c2c-8145-cafc2c5843df)


###  Prepare the etcd database for encryption at rest

Kubernetes uses etcd (A distributed key value store) to store variety of data which includes the cluster state, application configurations, and secrets. By default, the data that is being persisted to the disk is not encrypted.

- Generate the encryption key and encode it using base64
  
  ``ETCD_ENCRYPTION_KEY=$(head -c 64 /dev/urandom | base64)``

  ![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/1f391156-4730-4930-86af-d0308bf2826e)

  
### Send the encryption file to the Controller nodes using scp and a for loop

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/6da88896-7eae-463e-8854-25ea4bd76db5)

### You should have a a similar pane like below. You should be able to see all the files that have been sent to the nodes

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/f7d3b098-60df-47d6-953e-46da17a5f455)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/5ab41aa9-26ec-4045-a811-c0de6f8e74d7)

