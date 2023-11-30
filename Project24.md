## Building Elastic Kubernetes Service (EKS) With Terraform - Deploying and Packaging applications with Helm

This project seeks to solidify your skills by focusing more on best practice setup:

- You will use Terraform to create a Kubernetes EKS cluster and dynamically add scalable worker nodes
- You will deploy multiple applications using HELM
- You will experience more kubernetes objects and how to use them with Helm.
- You will improve upon your CI/CD skills with Jenkins

**Note:** Use Terraform version v1.0.2

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/240b5211-1550-4026-8c83-39dd7edb3bfe)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bcc0a529-d156-40a7-9ee0-b30b524c2aa1)

 
Open up a new directory on your laptop, and name it ``EKS`` Use AWS CLI to create an S3 bucket

Create a file – ``backend.tf`` Task for you, ensure the backend is configured for remote state in S3

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/20e7f605-dab5-40ba-930f-d128f70ef9c3)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/9066348e-3ec1-4344-9a34-26f66a8074fd)


Create a file – `network.tf` and provision Elastic IP for Nat Gateway, VPC, Private and public subnets.

### Reserve Elastic IP to be used in our NAT gateway

```
resource "aws_eip" "nat_gw_elastic_ip" {
vpc = true

tags = {
Name            = "${var.cluster_name}-nat-eip"
iac_environment = var.iac_environment_tag
}
}
```

### Create VPC using the official AWS module

```
module "vpc" {
source  = "terraform-aws-modules/vpc/aws"

name = "${var.name_prefix}-vpc"
cidr = var.main_network_block
azs  = data.aws_availability_zones.available_azs.names

private_subnets = [
# this loop will create a one-line list as ["10.0.0.0/20", "10.0.16.0/20", "10.0.32.0/20", ...]
# with a length depending on how many Zones are available
for zone_id in data.aws_availability_zones.available_azs.zone_ids :
cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) - 1)
]

public_subnets = [
# this loop will create a one-line list as ["10.0.128.0/20", "10.0.144.0/20", "10.0.160.0/20", ...]
# with a length depending on how many Zones are available
# there is a zone Offset variable, to make sure no collisions are present with private subnet blocks
for zone_id in data.aws_availability_zones.available_azs.zone_ids :
cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) + var.zone_offset - 1)
]

# Enable single NAT Gateway to save some money
# WARNING: this could create a single point of failure, since we are creating a NAT Gateway in one AZ only
# feel free to change these options if you need to ensure full Availability without the need of running 'terraform apply'
# reference: https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/2.44.0#nat-gateway-scenarios
enable_nat_gateway     = true
single_nat_gateway     = true
one_nat_gateway_per_az = false
enable_dns_hostnames   = true
reuse_nat_ips          = true
external_nat_ip_ids    = [aws_eip.nat_gw_elastic_ip.id]

# Add VPC/Subnet tags required by EKS
tags = {
"kubernetes.io/cluster/${var.cluster_name}" = "shared"
iac_environment                             = var.iac_environment_tag
}
public_subnet_tags = {
"kubernetes.io/cluster/${var.cluster_name}" = "shared"
"kubernetes.io/role/elb"                    = "1"
iac_environment                             = var.iac_environment_tag
}
private_subnet_tags = {
"kubernetes.io/cluster/${var.cluster_name}" = "shared"
"kubernetes.io/role/internal-elb"           = "1"
iac_environment                             = var.iac_environment_tag
}
}
```

`Note`: The tags added to the subnets is very important. The Kubernetes Cloud Controller Manager (cloud-controller-manager) and AWS Load Balancer Controller (aws-load-balancer-controller) needs to identify the cluster’s. To do that, it querries the cluster’s subnets by using the tags as a filter.

* For public and private subnets that use load balancer resources: each subnet must be tagged
```
Key: kubernetes.io/cluster/cluster-name
Value: shared
```
* For private subnets that use internal load balancer resources: each subnet must be tagged
```
Key: kubernetes.io/role/internal-elb
Value: 1
```
* For public subnets that use internal load balancer resources: each subnet must be tagged
```
Key: kubernetes.io/role/elb
Value: 1
```


Create a file – `variables.tf`

### create some variables
```
variable "cluster_name" {
type        = string
description = "EKS cluster name."
}
variable "iac_environment_tag" {
type        = string
description = "AWS tag to indicate environment name of each infrastructure object."
}
variable "name_prefix" {
type        = string
description = "Prefix to be used on each infrastructure object Name created in AWS."
}
variable "main_network_block" {
type        = string
description = "Base CIDR block to be used in our VPC."
}
variable "subnet_prefix_extension" {
type        = number
description = "CIDR block bits extension to calculate CIDR blocks of each subnetwork."
}
variable "zone_offset" {
type        = number
description = "CIDR block bits extension offset to calculate Public subnets, avoiding collisions with Private subnets."
}
```

Create a file – `data.tf` – This will pull the available AZs for use.
```
# get all available AZs in our region
data "aws_availability_zones" "available_azs" {
state = "available"
}
data "aws_caller_identity" "current" {} # used for accesing Account ID and ARN
```

create a file - __csi.tf__.

__The Container Storage Interface (CSI)__ provides a uniform interface for container orchestrators, such as Kubernetes, to interact with various storage solutions, encompassing both block and file storage types. A CSI driver serves as the intermediary that allows these orchestrators to connect with distinct storage systems or services, including cloud storage, network-attached storage (NAS), and storage area networks (SAN).
Read more about [__EKS add-on__](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html).

__Blocker:__ without installing the Amazon EBS CSI driver, Jenkins pod on Helm was stuck in "pending" state I had install this [Amazon EBS CSI driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html) to resolve the issue)

```
data "aws_iam_policy" "ebs_csi_policy" {
  arn = "arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy"
}

module "irsa-ebs-csi" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-assumable-role-with-oidc"
  version = "4.7.0"

  create_role                   = true
  role_name                     = "AmazonEKSTFEBSCSIRole-${var.cluster_name}"
  provider_url                  = module.eks_cluster.oidc_provider
  role_policy_arns              = [data.aws_iam_policy.ebs_csi_policy.arn]
  oidc_fully_qualified_subjects = ["system:serviceaccount:kube-system:ebs-csi-controller-sa"]
}

resource "aws_eks_addon" "ebs-csi" {
  cluster_name             = var.cluster_name
  addon_name               = "aws-ebs-csi-driver"
  service_account_role_arn = module.irsa-ebs-csi.iam_role_arn
  tags = {
    "eks_addon" = "ebs-csi"
    "terraform" = "true"
  }
}
```


Create a file – `eks.tf` and provision EKS cluster (Create the file only if you are not using your existing Terraform code. Otherwise you can simply append it to the main.tf from your existing code) [Read more about this module from the official documentation here](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/17.1.0) – Reading it will help you understand more about the rich features of the module.

```
module "eks_cluster" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 18.0"
  cluster_name    = var.cluster_name
  cluster_version = "1.22"
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  cluster_endpoint_private_access = true
  cluster_endpoint_public_access = true

  # Self Managed Node Group(s)
  self_managed_node_group_defaults = {
    instance_type                          = var.asg_instance_types[0]
    update_launch_template_default_version = true
  }
  self_managed_node_groups = local.self_managed_node_groups

  # aws-auth configmap
  create_aws_auth_configmap = true
  manage_aws_auth_configmap = true
  aws_auth_users = concat(local.admin_user_map_users, local.developer_user_map_users)
  tags = {
    Environment = "prod"
    Terraform   = "true"
  }
}
```


8. Create a file – `locals.tf` to create local variables. Terraform does not allow assigning variable to variables. There is good reasons for that to avoid repeating your code unecessarily. So a terraform way to achieve this would be to use locals so that your code can be kept __"Don't Repeat Yourself" (DRY)__

### render Admin & Developer users list with the structure required by EKS module
```
locals {
  admin_user_map_users = [
    for admin_user in var.admin_users :
    {
      userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${admin_user}"
      username = admin_user
      groups   = ["system:masters"]
    }
  ]
  developer_user_map_users = [
    for developer_user in var.developer_users :
    {
      userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${developer_user}"
      username = developer_user
      groups   = ["${var.name_prefix}-developers"]
    }
  ]

  self_managed_node_groups = {
    worker_group1 = {
      name = "${var.cluster_name}-wg"

      min_size      = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      desired_size      = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      max_size  = var.autoscaling_maximum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      instance_type = var.asg_instance_types[0].instance_type

      bootstrap_extra_args = "--kubelet-extra-args '--node-labels=node.kubernetes.io/lifecycle=spot'"

      block_device_mappings = {
        xvda = {
          device_name = "/dev/xvda"
          ebs = {
            delete_on_termination = true
            encrypted             = false
            volume_size           = 10
            volume_type           = "gp2"
          }
        }
      }

      use_mixed_instances_policy = true
      mixed_instances_policy = {
        instances_distribution = {
          spot_instance_pools = 4
        }

        override = var.asg_instance_types
      }
    }
  }
}
```



Add more variables to the `variables.tf` file

### create some variables
```
variable "admin_users" {
  type        = list(string)
  description = "List of Kubernetes admins."
}
variable "developer_users" {
  type        = list(string)
  description = "List of Kubernetes developers."
}
variable "asg_instance_types" {
  description = "List of EC2 instance machine types to be used in EKS."
}
variable "autoscaling_minimum_size_by_az" {
  type        = number
  description = "Minimum number of EC2 instances to autoscale our EKS cluster on each AZ."
}
variable "autoscaling_maximum_size_by_az" {
  type        = number
  description = "Maximum number of EC2 instances to autoscale our EKS cluster on each AZ."
}

```
Create a file – `variables.tfvars` to set values for variables.
```
cluster_name            = "tooling-app-eks"
iac_environment_tag     = "development"
name_prefix             = "darey-io-eks"
main_network_block      = "10.0.0.0/16"
subnet_prefix_extension = 4
zone_offset             = 8
```

### Ensure that these users already exist in AWS IAM. Another approach is that you can introduce an iam.tf file to manage users separately, get the data source and interpolate their ARN.

```
admin_users                    = ["darey", "solomon"]
developer_users                = ["leke", "david"]
asg_instance_types             = [ { instance_type = "t3.small" }, { instance_type = "t2.small" }, ]
autoscaling_minimum_size_by_az = 1
autoscaling_maximum_size_by_az = 10
```

Create file – `provider.tf`

```
provider "aws" {
  region = "us-west-1"
}
provider "random" {
}
```

Create a file – `variables.tfvars` to set values for variables.
```
cluster_name            = "tooling-app-eks"
iac_environment_tag     = "development"
name_prefix             = "darey-io-eks"
main_network_block      = "10.0.0.0/16"
subnet_prefix_extension = 4
zone_offset             = 8
```

### Ensure that these users already exist in AWS IAM. Another approach is that you can introduce an iam.tf file to manage users separately, get the data source and interpolate their ARN.

```
admin_users                              = ["dare", "solomon"]
developer_users                          = ["leke", "david"]
asg_instance_types                       = ["t3.small", "t2.small"]
autoscaling_minimum_size_by_az           = 1
autoscaling_maximum_size_by_az           = 10
autoscaling_average_cpu                  = 30
```

`Run terraform init`

### Run Terraform plan – Your plan should have an output
```
Plan: 41 to add, 0 to change, 0 to destroy.
```
Run Terraform apply

This will begin to create cloud resources, and fail at some point with the error
```

╷
│ Error: Post "http://localhost/api/v1/namespaces/kube-system/configmaps": dial tcp [::1]:80: connect: connection refused
│ 
│   with module.eks-cluster.kubernetes_config_map.aws_auth[0],
│   on .terraform/modules/eks-cluster/aws_auth.tf line 63, in resource "kubernetes_config_map" "aws_auth":
│   63: resource "kubernetes_config_map" "aws_auth" {
```



### To fix this problem

Append to the file data.tf
### Get EKS cluster info to configure Kubernetes and Helm providers
```
data "aws_eks_cluster" "cluster" {
  name = module.eks_cluster.cluster_id
}
data "aws_eks_cluster_auth" "cluster" {
  name = module.eks_cluster.cluster_id
}
```

Append to the file provider.tf
### Get EKS authentication for being able to manage k8s objects from terraform
```
provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  token                  = data.aws_eks_cluster_auth.cluster.token
}
```

### Run the init and plan again – This time you will see

```
  # module.eks-cluster.kubernetes_config_map.aws_auth[0] will be created
  + resource "kubernetes_config_map" "aws_auth" {
      + data = {
          + "mapAccounts" = jsonencode([])
          + "mapRoles"    = <<-EOT
                - "groups":
                  - "system:bootstrappers"
                  - "system:nodes"
                  "rolearn": "arn:aws:iam::696742900004:role/tooling-app-eks20210718113602300300000009"
                  "username": "system:node:{{EC2PrivateDNSName}}"
            EOT
          + "mapUsers"    = <<-EOT
                - "groups":
                  - "system:masters"
                  "userarn": "arn:aws:iam::696742900004:user/dare"
                  "username": "dare"
                - "groups":
                  - "system:masters"
                  "userarn": "arn:aws:iam::696742900004:user/solomon"
                  "username": "solomon"
                - "groups":
                  - "darey-io-eks-developers"
                  "userarn": "arn:aws:iam::696742900004:user/leke"
                  "username": "leke"
                - "groups":
                  - "darey-io-eks-developers"
                  "userarn": "arn:aws:iam::696742900004:user/david"
                  "username": "david"
            EOT
        }
      + id   = (known after apply)

      + metadata {
          + generation       = (known after apply)
          + labels           = {
              + "app.kubernetes.io/managed-by" = "Terraform"
              + "terraform.io/module"          = "terraform-aws-modules.eks.aws"
            }
          + name             = "aws-auth"
          + namespace        = "kube-system"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/c3305c8a-d758-4149-ba16-6bcc5d9b5e0e)


### Connect to eks cluster

``aws eks --region us-west-1 update-kubeconfig --name tooling-app-eks``

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bdb378db-4ce6-4b17-a774-90d4e10808b4)


### Deploy applications with Helm

Helm stands out as a widely favored tool for deploying resources into Kubernetes in the real world, primarily due to its capability to package deployments as cohesive units. This approach streamlines the process, in contrast to managing multiple individual YAML files, which can become cumbersome.

Essentially, a Helm chart acts as a blueprint for the necessary resources to operate an application within Kubernetes. It simplifies the process by consolidating various components such as deployments, services, volumes, configmaps, and more into a single, manageable entity. For instance, deploying a MySQL database becomes as straightforward as executing the command:

 ``helm install stable/mysql``

This command instructs Helm to ensure all essential resources are deployed correctly. Moreover, Helm offers the flexibility to customize deployments by adjusting variables. A practical example is activating a slave feature for MySQL, which enables the deployment of read-only replicas.

At its core, a Helm chart comprises a collection of YAML manifests detailing all the resources an application requires. Helm efficiently manages these resources, handling the creation of new resources in Kubernetes and the removal of outdated ones.


### To check Helm Version

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/535027c0-8bb1-4eb5-9e69-1ac698135f54)



### Deploy Jenkins with Helm

Before diving into creating custom Helm charts, it's beneficial to start by deploying tools using publicly available charts. One of the remarkable features of Helm is its ability to deploy pre-packaged applications from a public Helm repository with minimal configuration needed. A good example of this is Jenkins.

To access these packaged applications as Helm Charts:

1. Visit [Artifact Hub](https://artifacthub.io/packages/search), which is a comprehensive repository for finding Helm Charts for various applications.
1. Search for "Jenkins" to find the Helm Chart for it.
1. Add the Jenkins repository to your Helm setup. This step makes it straightforward to download and deploy Jenkins directly using Helm.

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/a58d4cbe-5968-41e4-a724-46e206ab4624)

Update helm repo

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/27e417d9-a282-47d0-a4e5-1a1baabc617a)

Install Jenkins chart

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/17e611b6-34d7-487c-817c-4127ace28e87)


Check the Helm deployment

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/4d73bb1d-ed12-409c-8be7-98c5bc944b9f)


Check the pods

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/1de33ee8-4db4-4893-97eb-c3dbdea233ec)

Describe the running pod

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/0f65c5fd-d5d2-4519-bfe4-4089236f5d63)


Check the logs of the running pod

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/2a2d2013-c1f0-44cb-b33f-de9a87051ceb)


This is because the pod has a __Sidecar container__ alongside with the Jenkins container. As you can see from the error output, there is a list of containers inside the pod __[jenkins config-reload]__ i.e jenkins and config-reload containers. The job of the config-reload is mainly to help Jenkins to reload its configuration without recreating the pod.

Therefore we need to let __kubectl__ know, which pod we are interested to see its log. 

Run the command to specify __jenkins__

`kubectl logs jenkins-0 -c jenkins`

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/5456df60-93eb-436d-bb5b-71d8a8bba7a0)


Accessing jenkins From UI

There are some commands that was provided on the screen when Jenkins was installed with Helm. Get the password to the admin user

`kubectl exec --namespace default -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/chart-admin-password && echo`

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/1c01e2ef-7d92-48a9-8881-87d0807430d7)

Use port forwarding to access Jenkins from the UI

`kubectl --namespace default port-forward svc/jenkins 8080:8080`

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/4b0fecbe-2419-491e-8836-dcd4753c8446)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/a4fa5a07-a04b-42b3-8b6d-a445e677900d)


### TASK
Use Helm to to install:

### Prometheus

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```
helm repo update
```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/926734e0-088e-407c-9d77-30c4ee9beb59)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/f58d2840-b730-4aca-94bc-8db028d0a243)


### Grafana installation

```helm repo add grafana https://grafana.github.io/helm-charts```

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/e92e7db8-5538-4864-9567-93a47f797e75)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/5610bff2-4c63-495b-8cb9-156b71b396bd)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/58e6f5c5-4797-421c-860f-36193eef636a)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/43f22388-3296-4600-b2a3-e2b755ea2f73)


### Elasticsearch ELK using ECK
