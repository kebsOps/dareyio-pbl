## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

After you have built an AWS infrastructure for 2 websites manually in [Project 15](https://github.com/kebsOps/dareyio-pbl/blob/main/project15.md), it is time to automate the process using Terraform.

Let us start building the same set up with the power of Infrastructure as Code (IaC)

<img width="657" alt="image" src="https://user-images.githubusercontent.com/10085348/189526590-6befa92b-138a-4ba7-adc1-09747aeca0cd.png">


### Prerequisites before you begin writing Terraform code

- Create an IAM user, name it terraform (ensure that the user has only programatic access to your AWS account) and grant this user ``AdministratorAccess`` permissions.

<img width="854" alt="image" src="https://user-images.githubusercontent.com/10085348/189527396-952d783b-4cf7-4152-8ff1-6d683a13797e.png">

- Copy the secret access key and access key ID. Save them in a notepad temporarily.
- Configure programmatic access from your workstation to connect to AWS using the access keys copied above and a Python SDK (boto3). You must have Python 3.6 or higher on your workstation.


_For easier authentication configuration – use AWS CLI with aws configure command._


### The secrets of writing quality Terraform code

The secret recipe of a successful Terraform projects consists of:

- Your understanding of your goal (desired AWS infrastructure end state)
- Your knowledge of the IaC technology used (in this case – Terraform)
- Your ability to effectively use up to date Terraform documentation [here](https://www.terraform.io/language)


Create an S3 bucket to store Terraform state file. You can name it something like ``<yourname>-dev-terraform-bucket``

<img width="1176" alt="image" src="https://user-images.githubusercontent.com/10085348/189527845-f1206aa1-8995-4b94-a799-c6bd18e18df7.png">

<img width="386" alt="image" src="https://user-images.githubusercontent.com/10085348/189528132-cc95df9e-58b1-4eaa-8cb7-1a609e2b5909.png">

When you have configured authentication and installed boto3, make sure you can programmatically access your AWS account by running following commands in ``>python:``
```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```

<img width="785" alt="image" src="https://user-images.githubusercontent.com/10085348/189528184-ff896c8d-c636-431e-9937-7875294e00e3.png">

Open your Visual Studio Code and:

- Create a folder called ``PBL``
- Create a file in the folder, name it ``main.tf``

Your setup should look like this:

<img width="1501" alt="image" src="https://user-images.githubusercontent.com/10085348/189529628-7afbe83f-33a1-4cdb-9cae-d2192dacfbee.png">


## Provider and VPC resource section

Set up Terraform CLI as per this [instruction](https://learn.hashicorp.com/tutorials/terraform/install-cli)

- Add AWS as a provider, and a resource to create a VPC in the ``main.tf`` file.
- Provider block informs Terraform that we intend to build infrastructure within AWS.
- Resource block will create a VPC.

<img width="1083" alt="image" src="https://user-images.githubusercontent.com/10085348/189530197-1cac1855-47c1-418b-8e51-7ca6a42a4b5f.png">

- The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by ``providers`` and ``provisioners``. At this stage, we only have provider in our ``main.tf`` file.
So, Terraform will just download plugin for AWS provider.
Lets accomplish this with ``terraform init`` command as seen in the below:

<img width="751" alt="image" src="https://user-images.githubusercontent.com/10085348/189530953-2f5a16c1-e16d-4805-8180-b4abd4bcfac6.png">

**Observations:

Notice that a new directory has been created: ``.terraform\....`` This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that you must execute terraform init again, to download them.
Moving on, let us create the only resource we just defined. ``aws_vpc``. But before we do that, we should check to see what terraform intends to create before we tell it to go ahead and create it.

Run ``terraform plan``

<img width="1016" alt="image" src="https://user-images.githubusercontent.com/10085348/189531502-c0da3250-44fd-487b-84fa-38ee93590b3b.png">

Then, if you are happy with changes planned, execute ``terraform apply``

<img width="1088" alt="image" src="https://user-images.githubusercontent.com/10085348/189531652-28665bb2-f575-4ff2-be21-a52f79ab467b.png">

**Observation

- A new file is created ``terraform.tfstate`` This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.

**PBL folder structure

<img width="176" alt="image" src="https://user-images.githubusercontent.com/10085348/189531767-fd3bc9f2-a66b-4fbb-8252-a58b638f9c16.png">

- If you also observed closely, you would realise that another file gets created during planning and apply. But this file gets deleted immediately. ``terraform.tfstate.lock.info`` This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts.


### Create Subnet Resources

According to the architectural design 6 subnets is required: 
- 2 public
- 2 private for webservers 
- 2 private for data layer

- Create 2 public subnets by entering the following codes:

<img width="376" alt="image" src="https://user-images.githubusercontent.com/10085348/189532666-2307b8eb-753e-46fa-b086-2915e0e632cb.png">
 
We are creating 2 subnets, therefore declaring 2 resource blocks – one for each of the subnets.

We are using the ``vpc_id`` argument to interpolate the value of the VPC id by setting it to ``aws_vpc.main.id``. This way, Terraform knows inside which VPC to create the subnet.

Run ``terraform plan`` and ``terraform apply``

<img width="820" alt="image" src="https://user-images.githubusercontent.com/10085348/189532941-3ee09bc9-7667-442b-bd00-cf7080206648.png">

<img width="726" alt="image" src="https://user-images.githubusercontent.com/10085348/189533009-eab9453f-2af9-4543-8d3f-06924070f43d.png">

<img width="629" alt="image" src="https://user-images.githubusercontent.com/10085348/189533029-80cafeeb-2e33-4c5a-8a6e-aadecbf923d2.png">



## Observations:

Hard coded values: Remember our best practice hint from the beginning? Both the ``availability_zone`` and ``cidr_block`` arguments are hard coded. We should always endeavour to make our work dynamic.

Multiple Resource Blocks: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a count argument.
Now let us improve our code by refactoring it.

_First, destroy the current infrastructure. Since we are still in development, this is totally fine. Otherwise, DO NOT DESTROY an infrastructure that has been deployed to production._

To destroy whatever has been created run ``terraform destroy command``, and type ``yes`` after evaluating the plan.
 
 <img width="1160" alt="image" src="https://user-images.githubusercontent.com/10085348/189533222-4dbbaf3b-90bc-4000-a12b-034012ce5c58.png">

<img width="1078" alt="image" src="https://user-images.githubusercontent.com/10085348/189533252-1fe0208b-a210-43ad-8cd0-c771fd953f87.png">


### Fixing The Problems By Code Refactoring

**- Fixing Hard Coded Values**: Lets introduce variables, and remove hard coding

 - Starting with the provider block, declare a variable named ``region``, give it a default value, and update the provider section by referring to the declared variable.
 
```
    variable "region" {
        default = "eu-west-1"
    }

    provider "aws" {
        region = var.region
    }
```

Do the same to `cidr` value in the vpc block, and all the other arguments

```
   variable "region" {
        default = "eu-west-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_hostnames
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
```

- Fixing multiple resource blocks: This is where things become a little tricky. It’s not complex, we are just going to introduce some interesting concepts. Loops & Data sources

Terraform has a functionality that allows us to pull data which exposes information to us. For example, every region has Availability Zones (AZ). Different regions have from 2 to 4 Availability Zones. With over 20 geographic regions and over 70 AZs served by AWS, it is impossible to keep up with the latest information by hard coding the names of AZs. Hence, we will explore the use of Terraform’s Data Sources to fetch information outside of Terraform. In this case, from AWS

Let us fetch Availability zones from AWS, and replace the hard coded value in the subnet’s ``availability_zone`` section.

 # Get list of availability zones
        ```
        data "aws_availability_zones" "available" {
        state = "available"
        }
        ```
  To make use of this new `data` resource, we will need to introduce a `count` argument in the subnet block: Something like this
  
  ```
     # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
  ```
 
 Let us quickly understand what is going on here.

The count tells us that we need 2 subnets. Therefore, Terraform will invoke a loop to create 2 subnets.
The data resource will return a list object that contains a list of AZs. Internally, Terraform will receive the data like this:

 ```["eu-west-1a", "eu-west-1b"]```
 
 Each of them is an index, the first one is index 0, while the other is index 1. If the data returned had more than 2 records, then the index numbers would continue to increment.

Therefore, each time Terraform goes into a loop to create a subnet, it must be created in the retrieved AZ from the list. Each loop will need the index number to determine what AZ the subnet will be created. That is why we have ``data.aws_availability_zones.available.names[count.index]`` as the value for availability_zone. When the first loop runs, the first index will be 0, therefore the AZ will be ``eu-west-1a``. The pattern will repeat for the second loop.

But we still have a problem. If we run Terraform with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because we still have cidr_block hard coded. The same ``cidr_block`` cannot be created twice within the same VPC. So, we have a little more work to do.

### Let’s make cidr_block dynamic.

We will introduce a function **cidrsubnet()** to make this happen. It accepts 3 parameters. Let us use it first by updating the configuration, then we will explore its internals.

  ### Create public subnet
  
  ```
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```

A closer look at cidrsubnet – this function works like an algorithm to dynamically create a subnet CIDR per AZ. Regardless of the number of subnets created, it takes care of the cidr value per subnet.

### The final problem to solve is removing hard coded count value

- If we cannot hard code a value we want, then we will need a way to dynamically provide the value based on some input. Since the data resource returns all the `AZs` within a region, it makes sense to count the number of `AZs` returned and pass that number to the count argument.

- To do this, we can introuduce `length()` function, which basically determines the length of a given list, map, or string.

Since `data.aws_availability_zones.available.names` returns a list like `["eu-central-1a", "eu-central-1b", "eu-central-1c"]` we can pass it into a lenght function and get number of the AZs.

`length(["eu-central-1a", "eu-central-1b", "eu-central-1c"])`


Now we can simply update the public subnet block like this

```
# Create public subnet
    resource "aws_subnet" "public" { 
        count                   = length(data.aws_availability_zones.available.names)
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```

**Observations:**

What we have now, is sufficient to create the subnet resource required. But if you observe, it is not satisfying our business requirement of just 2 subnets. The length function will return number 3 to the count argument, but what we actually need is 2.
Now, let us fix this.

Declare a variable to store the desired number of public subnets, and set the default value

```
variable "preferred_number_of_public_subnets" {
  default = 2
}
```


Next, update the count argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the lenght function. See how that is presented below:

```
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```

Now lets break it down:

- The first part `var.preferred_number_of_public_subnets == null` checks if the value of the variable is set to null or has some value defined.

- The second part `?` and `length(data.aws_availability_zones.available.names)` means, if the first part is true, then use this. In other words, if preferred number of public subnets is `null` (or not known) then set the value to the data returned by `lenght` function.

The third part : and  `var.preferred_number_of_public_subnets` means, if the first condition is `false`, i.e `preferred number of public subnets` is `not null` then set the value to whatever is definied in `var.preferred_number_of_public_subnets`


### Now the entire configuration should now look like this

<img width="1135" alt="image" src="https://user-images.githubusercontent.com/10085348/189735037-66b293ae-c52d-47e7-9016-adab90bc4f3d.png">

**blocker:**

- `available` was mispelt in the # Get list of availability zones code section above (state = "available"), this has been updated:

<img width="343" alt="image" src="https://user-images.githubusercontent.com/10085348/189739656-e297152c-745e-4e43-afe9-338503125c98.png">

```

# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}
```

- `enable_classiclink_dns_support` was omitted earlier but now updated

```
 variable "enable_classiclink_dns_support" {
 default = "false"
}
```
<img width="423" alt="image" src="https://user-images.githubusercontent.com/10085348/189742681-bae8bd06-1ad9-4033-90b0-c03c3ad4514f.png">


## Introducing variables.tf & terraform.tfvars

Instead of havng a long lisf of variables in main.tf file, we can actually make our code a lot more readable and better structured by moving out some parts of the configuration content to other files.

- We will put all variable declarations in a separate file And provide non default values to each of them
- Create a new file and name it `variables.tf`
- Copy all the variable declarations into the new file

**main.tf**

<img width="1174" alt="image" src="https://user-images.githubusercontent.com/10085348/189738831-cef2ca15-3b9f-41d6-895b-df38d092dd35.png">

**variables.tf**

<img width="705" alt="image" src="https://user-images.githubusercontent.com/10085348/189742243-c891798d-9f73-4ab3-9235-1a2abde27c2e.png">

- Create another file, name it `terraform.tfvars`
- Set values for each of the variables

**terraform.tfvars**

<img width="426" alt="image" src="https://user-images.githubusercontent.com/10085348/189737097-28d46d65-20bb-412f-9d4d-5a70ca321356.png">


You should also have this file structure in the PBL folder


<img width="378" alt="image" src="https://user-images.githubusercontent.com/10085348/189736737-9f00dbac-24d1-4eed-87f8-d00c22fdca8a.png">

Run `terraform plan` and ensure everything works

<img width="1124" alt="image" src="https://user-images.githubusercontent.com/10085348/189743073-14f5fb05-316f-4acf-8c0b-494127bdaf75.png">

<img width="1253" alt="image" src="https://user-images.githubusercontent.com/10085348/189743152-96d60e7d-c7c3-4a21-8278-6354bc9ffbf2.png">

Run `terraform apply -auto-approve` if satisfied

<img width="1221" alt="image" src="https://user-images.githubusercontent.com/10085348/189743540-21ef5fa7-9c79-4259-bbf1-c9210636b430.png">

<img width="845" alt="image" src="https://user-images.githubusercontent.com/10085348/189743639-0516390f-a1cf-4307-b181-e63a46e69a5c.png">


