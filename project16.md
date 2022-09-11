## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

After you have built AWS infrastructure for 2 websites manually in [Project 15](https://github.com/kebsOps/dareyio-pbl/blob/main/project15.md), it is time to automate the process using Terraform.

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




