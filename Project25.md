## Deploying and Packaging applications into Kubernetes with Helm

In [project 24](https://github.com/kebsOps/dareyio-pbl/blob/main/Project24.md), you started experiencing helm as a tool used to deploy an application into Kubernetes. You probably also tried installing more tools apart from Jenkins.

In this project, you will experience deploying more DevOps tools, get familiar with some of the real world issues faced during such deployments and how to fix them. You will learn how to tweak helm values files to automate the configuration of the applications you deploy. Finally, once you have most of the DevOps tools deployed, you will experience using them and relate with the DevOps cycle and how they fit into the entire  ecosystem.

Our focus will be on the. 

- Artifactory
- Ingress Controllers
- Cert-Manager

Then

- Prometheus
- Grafana
- Elasticsearch ELK using [ECK](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-install-helm.html)

For the tools that require paid license, don't worry, you will also learn how to get the license for free and have true experience exactly how they are used in the real world.

Lets start first with Artifactory. What is it exactly?

Artifactory is part of a suit of products from a company called [Jfrog](https://jfrog.com/). Jfrog started out as an artifact repository where software binaries in different formats are stored. Today, Jfrog has transitioned from an artifact repository to a DevOps Platform that includes CI and CD capabilities. This has been achieved by offering more products in which **Jfrog Artifactory** is part of. Other offerings include 
      
  - __JFrog Pipelines__ -  a CI-CD product that works well with its Artifactory repository. Think of this product as an alternative to Jenkins.
  - __JFrog Xray__ - a security product that can be built-into various steps within a JFrog pipeline. Its job is to scan for security vulnerabilities in the stored artifacts. It is able to scan all dependent code.

In this project, the requirement is to use Jfrog Artifactory as a private registry for the organisation's Docker images and Helm charts. This requirement will satisfy part of the company's corporate security policies to never download artifacts directly from the public into production systems. We will eventually have a CI pipeline that initially pulls public docker images and helm charts from the internet, store in artifactory and scan the artifacts for security vulnerabilities before deploying into the corporate infrastructure. Any found vulnerabilities will immediately trigger an action to quarantine such artifacts.

Lets get into action and see how all of these work.

Create the cluster

``eksctl create cluster --name kebsOps-eks-tooling --region us-west-1 --nodegroup-name worker --node-type t3.medium --nodes 2``

Create kubeconfig file using awscli and connect to eks cluster

``aws eks --region us-west-1 update-kubeconfig --name kebsOps-eks-tooling``

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/58f9530a-83ff-42f4-8638-30f09fa00cc1)


__Create EBS-CSI Driver for the Cluster__

The Amazon Elastic Block Store (EBS) Container Storage Interface (CSI) driver is a vital component for Kubernetes clusters that utilize Amazon EBS for persistent storage. It facilitates a seamless connection between Kubernetes and EBS, enabling automated and efficient management of EBS volumes for containerized applications. Here's why the EBS CSI driver is essential:

1. __Dynamic Provisioning:__ The EBS CSI driver allows for the automatic creation and configuration of EBS volumes within Kubernetes, streamlining storage provisioning and reducing manual effort.
1. __Automated Volume Attachment:__ It automatically attaches and detaches EBS volumes to Kubernetes nodes based on pod requirements, ensuring containers have the necessary storage without manual processes.
1. __Volume Lifecycle Management:__ This driver manages EBS volumes' entire lifecycle, including creation, deletion, resizing, and snapshotting, providing a comprehensive storage management solution within Kubernetes.
1. __Simplified Storage Management:__ By separating the storage interface from the Kubernetes controller manager, the EBS CSI driver simplifies storage management and reduces the complexity of Kubernetes operations.
1. __Enhanced Storage Flexibility:__ It supports various EBS volume configurations, allowing customization of storage to meet specific application needs.
1. __Integration with Kubernetes Ecosystem:__ The driver integrates with Kubernetes PersistentVolumes, PersistentVolumeClaims, and StorageClasses, ensuring compatibility with existing Kubernetes storage workflows.
The EBS CSI driver is crucial for Kubernetes clusters to effectively utilize EBS for persistent storage, offering simplified management, automated operations, and flexible storage options.


__Installing EBS CSI Driver__

Click on the setup link [__EBS CSI add-on__](https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html)

Use the command below to check platform version.

``$ aws eks describe-addon-versions --addon-name aws-ebs-csi-driver``

<img width="1398" alt="Screenshot 2023-12-06 at 10 39 33" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/6b81d6d3-6071-4039-a2fd-620adc9dbe4a">

You may already possess an AWS IAM OpenID Connect (OIDC) provider associated with your cluster. To verify its presence or to create one, examine the OIDC issuer URL connected to your cluster. Having an IAM OIDC provider is essential for employing IAM roles in conjunction with service accounts. To establish an IAM OIDC provider for your cluster, you can utilize tools like __eksctl or the AWS Management Console.__

__To create an IAM OIDC identity provider for your cluster with eksctl__

Determine the OIDC issuer ID for your cluster.

Retrieve your cluster's OIDC issuer ID and store it in a variable.

``cluster_name=kebsOps-eks-tooling``

``oidc_id=$(aws eks describe-cluster --name $cluster_name --region us-west-1 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)``

``echo $oidc_id``

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bdbacfc0-ea6d-46e8-924b-d5e9051b58ca)

check to see if there's an IAM OIDC provider in your aws account that matches your cluster's issuer ID.

`$ aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4`

If output is returned, then you already have an IAM OIDC provider for your cluster and you can skip the next step. If no output is returned, then you must create an IAM OIDC provider for your cluster.

In our case, no output was returned.

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/d4c2971f-5551-4695-afb6-7c9b93933f1c)

Create an IAM OIDC identity provider for your EKS cluster with the command below

``eksctl utils associate-iam-oidc-provider --cluster $cluster_name --region us-west-1 --approve``

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/5481e163-93c6-445b-88d6-5e0b8047b016)


Configure a Kubernetes service account to assume an IAM role

Create a ```aws-ebs-csi-driver-trust-policy.json``` file that includes the permissions for the AWS services

```
 cat >aws-ebs-csi-driver-trust-policy.json <<EOF                                                 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::982035331548:oidc-provider/oidc.eks.us-west-1.amazonaws.com/id/F461F9CFCFB620CEAE0FA86BD5C6201E"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-west-1.amazonaws.com/id/F461F9CFCFB620CEAE0FA86BD5C6201E:aud": "sts.amazonaws.com",
          "oidc.eks.us-west-1.amazonaws.com/id/F461F9CFCFB620CEAE0FA86BD5C6201E:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
        }
      }
    }
  ]
}
EOF
```

This trust policy enables the EBS CSI driver to take on the role linked to the given OIDC provider, allowing it to access AWS resources acting on behalf of the EBS CSI controller service account.


Create a __AmazonEKS_EBS_CSI_DriverRole__ role

```
aws iam create-role \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --assume-role-policy-document file://"aws-ebs-csi-driver-trust-policy.json"
```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/00ae633a-94ad-4409-b079-f4285fc441d9)


Attach AWS managed policy to the role.

```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --role-name AmazonEKS_EBS_CSI_DriverRole
```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/7529f348-d913-4067-a710-4b18ff689102)


Add the Amazon EBS CSI add-on using the AWS CLI command below

```
aws eks create-addon --cluster-name $cluster_name --addon-name aws-ebs-csi-driver \
  --service-account-role-arn arn:aws:iam::982035331548:role/AmazonEKS_EBS_CSI_DriverRole --region us-west-1
```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/5b16a347-460c-484c-b982-af53a52ca552)


Use the command below to verify the EBS-CSI Driver installation in the kube-system namespace 

``kubectl get pods -n kube-system``

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/d7f91890-b687-4160-b9aa-83c1e367e43c)


## Deploy Jfrog Artifactory into Kubernetes

The best approach to easily get Artifactory into kubernetes is to use helm.

Search for an official helm chart for Artifactory on [Artifact Hub](https://artifacthub.io/)
   
<img width="1479" alt="Screenshot 2023-12-05 at 15 34 01" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/d6b2eb3c-e542-422b-9079-834bc4f697c1">

Click on install to see installation instructions.

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/684fb618-1cd8-4586-95af-d988e81fe5bb)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/15ca6c29-2a2f-423b-b288-f92869e30551)

Add the jfrog remote repository on your laptop/computer

`helm repo add jfrog https://charts.jfrog.io`

Create a namespace called tools where all the tools for DevOps will be deployed. 

`kubectl create ns tools`

Update the helm repo index on your local machine/laptop

`helm repo update`

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/d466aa7f-de45-41ed-9c0a-99a4d0176613)


Install artifactory in the namespace __tools__

`helm upgrade --install artifactory jfrog/artifactory --version 107.71.5 -n tools`

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/26403177-1c1d-4e60-8f13-f2ae12bb95b0)

Here, the __upgrade --install flag__ has been chosen over the usual ``helm install artifactory jfrog/artifactory`` command. This practice is superior, especially when creating CI pipelines with Helm, as it allows Helm to upgrade an existing installation or perform a fresh install if none exists. This technique guarantees that the command is error-proof, as it automatically determines whether to upgrade or install anew.

__Getting the Artifactory URL__

The artifactory helm chart comes bundled with the Artifactory software, a PostgreSQL database and an Nginx proxy which it uses to configure routes to the different capabilities of Artifactory. Getting the pods after some time, you should see something like the below:


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/b37b5bd7-7e3e-41d0-957a-57262a162fe8)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/0e7c1fd4-41d3-4a1a-99dc-3d0c1325dcd8)


Each of the deployed application have their respective services. This is how you will be able to reach either of them.
Notice that, the Nginx Proxy has been configured to use the service type of LoadBalancer. Therefore, to reach Artifactory, we will need to go through the Nginx proxy's service. Which happens to be a load balancer created in the cloud provider. Run the ``kubectl`` command to retrieve the Load Balancer URL.

``k get svc -n tools``

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/74f1ea5f-2c5d-4f10-b2fc-ed8ccb6482e5)

Use artifactory loadbalancer URL to access artifactory via our web browser

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/c57a3fb7-1bca-4e8e-af5b-155ab4bf421e)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bc3f9a09-a5d4-45a5-abcf-260f2b458ef0)




