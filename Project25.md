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


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/aaf83ad4-6260-4ce0-a896-b195004ac2e2)


__How the Nginx URL for Artifactory is configured in Kubernetes__

How did Helm configure the URL in kubernetes?

Helm uses the __values.yaml__ file to set every single configuration that the chart has the capability to configure. The best place to get started with an off the shelve chart from __artifacthub.io__ is to get familiar with the __DEFAULT VALUES__ section on Artifact hub.

<img width="1456" alt="Screenshot 2023-12-06 at 14 57 23" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/8308f0ab-d7c7-433d-b592-8eb8cc76761d">

Here you can search for key and value pair

<img width="1471" alt="Screenshot 2023-12-06 at 14 59 59" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/458563e7-cbdd-4a36-99a8-c051075c95be">

For example, when you type nginx in the search bar, it shows all the configured options for the nginx proxy.

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/93c07092-7d88-46f6-9465-e039e2b1223f)

selecting nginx.enabled from the list will take you directly to the configuration in the YAML file.

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/269430f1-698d-48d6-8833-bb36044443b8)


Search for nginx.service and select nginx.service.type

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/f82be144-891e-40f8-991b-1ceac3178d42)

To work directly with the `values.yaml` file, you can download the file locally by clicking on the download icon.

<img width="1462" alt="Screenshot 2023-12-06 at 15 09 25" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/e7d62337-fc1c-4b9f-b294-f7df552c39cc">

### Is the Load Balancer Service type the Ideal configuration option to use in the Real World?

Using the Load Balancer service type in Kubernetes is a straightforward method for exposing applications externally. However, provisioning individual load balancers for each application can be costly and complex, especially with a large number of deployments.

A more efficient alternative is employing [Kubernetes Ingress documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/), which necessitates setting up an [__Ingress Controller__](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). The key advantage here is the ability to utilize a single load balancer across multiple applications, optimizing cloud costs and simplifying management.

For now, we will leave artifactory, move on to the next phase of configuration (Ingress, DNS(Route53) and Cert Manager), and then return to Artifactory to complete the setup so that it can serve as a private docker registry and repository for private helm charts.


### Deploying Ingress Controller and managing Ingress Resources

An ingress is an API object that manages external access to the services in a kubernetes cluster. It is capable to provide load balancing, SSL termination and name-based virtual hosting. In otherwords, Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
Here is a simple example where an Ingress sends all its traffic to one Service:

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bd54dcf0-63df-42e7-9fab-a7c7490d66fc)

An ingress resource for Artifactory would look like this:

```
cat <<EOF > artifactory-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: artifactory
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.artifactory.sandbox.svc.toolingkb.xyz"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: artifactory
            port:
              number: 8082
EOF
```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/f703b3bc-bc2b-4875-b024-e38f3865bf99)


- An Ingress needs ``apiVersion``, ``kind``, ``metadata`` and ``spec`` fields
- The name of an Ingress object must be a valid DNS subdomain name
- Ingress frequently uses annotations to configure some options depending on the Ingress controller.
- Different Ingress controllers support different annotations. Therefore it is important to be up to date with the ingress controller's specific documentation to know 
what annotations are supported.
- It is recommended to always specify the ingress class name with the spec ingressClassName: nginx. This is how the Ingress controller is selected, especially when there are multiple configured ingress controllers in the cluster.
- The domain ``toolingkb.xyz`` should be replaced with your own domain.

Create ingress resource in the __tools__ namespace

`kubectl apply -f artifactory-ingress.yaml -n tools`

Get ingress resource

`kubectl get ingress -n tools`

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/e7145b2f-7c71-47d3-b8d7-ff532cb13f0c)

Now, take note of the following:

__CLASS__ - The nginx controller class name nginx

__HOSTS__ - The hostname to be used in the browser tooling.artifactory.sandbox.svc.toolingkb.xyz

__ADDRESS__ - The loadbalancer address that was created by the ingress controller


### Configure DNS

If anyone were to visit the tool, it would be very inconvenient sharing the long load balancer address. Ideally, you would create a DNS record that is human readable and can direct request to the balancer. This is exactly what has been configured in the ingress object - host: __"tooling.artifactory.sandbox.svc.toolingkb.xyz"__ but without a DNS record, there is no way that host address can reach the load balancer.

The __sandbox.svc.toolingkb.xyz__ part of the domain is the configured __HOSTED ZONE__ in AWS. So you will need to configure Hosted Zone in AWS console or as part of your infrastructure as code using terraform.

You must have purchased a domain name from a domain provider and configured the nameservers.

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/358c7686-4618-4103-a342-46280b9473ef)

Ensure to use the Nameservers provided in your hosted zone when configuring the Nameservers with your DNS provider, like GoDaddy, Namecheap, or similar services.

<img width="1468" alt="Screenshot 2023-12-06 at 16 20 56" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/bde13a05-e984-4b98-ae07-3362c5c3763d">

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/6f5ceb7d-8b76-4f03-854b-9ec15cab76cb)

