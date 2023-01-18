## Automate Infrastructure With IaC using Terraform. Part 4 – Terraform Cloud

In this project, we are going to run Terraform codes in Terraform cloud console. 
AMI is built differently with packer while Ansible is used to configure the infrastructure after its been provisioned by Terraform.


### Migrating your .tf codes to Terraform Cloud

***Some Prerequisties***: 
- Rename ``terraform.tfvars`` file to ``terraform.auto.tfvars`` to enable terraform cloud configure terraform variables automatically

- set your **AWS_ACCESS_ID** and **AWS_SECRET_ACCESS_KEY** as environment variables on terraform cloud to enable it have access to your AWS environment

### Run and apply configuration

<img width="1476" alt="image" src="https://user-images.githubusercontent.com/10085348/211000612-f3f9d753-599f-45c0-b8ab-e5e80151e370.png">

<img width="1239" alt="image" src="https://user-images.githubusercontent.com/10085348/211000773-3cd1299f-0af4-48be-aeca-c180ad01e470.png">

### Ansible

We make use of Ansible dynamic inventory to get IP addresses of our ec2-instances created based on their tag names and then run the required role on each server

<img width="730" alt="image" src="https://user-images.githubusercontent.com/10085348/212323007-b15694c2-fcb6-425a-ba4e-e8b16041ecfc.png">

Running ansible playbook

<img width="1004" alt="image" src="https://user-images.githubusercontent.com/10085348/212325573-5f24a26b-8b5d-40d7-b125-6fb97c2efc2e.png">


<img width="1211" alt="image" src="https://user-images.githubusercontent.com/10085348/212328264-6a318103-cf10-4380-854a-3ef34d896919.png">


<img width="1234" alt="image" src="https://user-images.githubusercontent.com/10085348/212329443-da05db80-6da6-4aad-9b51-3127e96d01d7.png">

[Link to ansible roles repo](https://github.com/kebsOps/ANSIBLE-DEPLOY-PBL-19)


### Checking All HealthCheck Status for all Target groups

***Nginx Target***

<img width="1033" alt="image" src="https://user-images.githubusercontent.com/10085348/212357408-668cff5e-c11a-4885-9968-ee8b88a0f551.png">


***Tooling Target***

<img width="1039" alt="image" src="https://user-images.githubusercontent.com/10085348/212357683-1dab7ae6-832c-4d45-b3de-9e4494467076.png">

***Wordpress Target***

<img width="1043" alt="image" src="https://user-images.githubusercontent.com/10085348/212357883-212d381f-79c9-4a50-bf5f-8a9f5ab89b43.png">


## Tooling app

<img width="1290" alt="image" src="https://user-images.githubusercontent.com/10085348/212356545-217de468-f761-4807-9d28-705730f4469e.png">

## Wordpress app

<img width="1279" alt="image" src="https://user-images.githubusercontent.com/10085348/212353269-30c5fd65-4b2a-4460-b50e-9ab8829c60a8.png">





## Practice Task № 1

- Configure 3 branches in your terraform-cloud repository for ``dev, test, prod environments``


<img width="867" alt="Screenshot 2023-01-16 at 13 45 36" src="https://user-images.githubusercontent.com/10085348/212681484-686ebff2-4501-45dd-b8a2-c4a827ffb121.png">


- Make necessary configuration to trigger runs automatically only for ``dev environment``

<img width="1259" alt="Screenshot 2023-01-16 at 13 32 44" src="https://user-images.githubusercontent.com/10085348/212679463-110584ca-960b-42be-92b3-854006106bc1.png">

- Create an Email and Slack notifications for certain events (e.g. started plan or errored run) and test it

  - Email notifications:

<img width="1206" alt="Screenshot 2023-01-16 at 13 39 23" src="https://user-images.githubusercontent.com/10085348/212681013-1e16b3df-9e32-48ce-8143-c25fcd2716fb.png">


  - Slack notifications:

![image](https://user-images.githubusercontent.com/10085348/212682326-6b5bad8e-bce2-4422-9cb9-e4b690fa9b97.png)


- Apply destroy from Terraform Cloud web console

<img width="1240" alt="image" src="https://user-images.githubusercontent.com/10085348/212358395-e2197809-4c43-4eec-8737-c6a01ccf56cf.png">

## Practice Task № 2 Working with Private repository

Create a simple Terraform repository (you can clone one from [here](https://github.com/hashicorp/learn-private-module-aws-s3-webapp) that will be your module

Import the module into your private registry

<img width="1230" alt="image" src="https://user-images.githubusercontent.com/10085348/213127181-ec72a5f7-9bd4-4eb1-aa31-078e20736aee.png">


Create a configuration that uses the module

<img width="613" alt="image" src="https://user-images.githubusercontent.com/10085348/213125649-cc5a549f-db3a-4f9c-ba92-299819d37263.png">


<img width="828" alt="image" src="https://user-images.githubusercontent.com/10085348/213125760-dc1fbe6b-ad30-4d35-8067-556d6c2c8d36.png">


<img width="533" alt="image" src="https://user-images.githubusercontent.com/10085348/213125836-56a43a8e-4421-49c1-8826-739bbe5cbb18.png">

[Link to repo](https://github.com/kebsOps/test-registry)

Create a workspace for the configuration

<img width="1032" alt="image" src="https://user-images.githubusercontent.com/10085348/213126606-60231857-c949-491a-a229-7c14c92a6b3c.png">



Deploy the infrastructure

<img width="1101" alt="image" src="https://user-images.githubusercontent.com/10085348/213118168-99052422-c08c-4f94-a106-fe93350180e6.png">



<img width="1008" alt="image" src="https://user-images.githubusercontent.com/10085348/213123909-2a6b598b-6c60-4f64-884d-426fee4f3a3d.png">


App deployed

<img width="1235" alt="image" src="https://user-images.githubusercontent.com/10085348/213124195-2a2f8408-55ff-4081-b1c5-2e838220102e.png">



Destroy Resources:


<img width="946" alt="image" src="https://user-images.githubusercontent.com/10085348/213124513-77eb228b-24ce-4bbf-bfaa-dc58c3615f7f.png">




<img width="1021" alt="image" src="https://user-images.githubusercontent.com/10085348/213124921-9239b120-e906-4492-96b2-f5077300cbc2.png">
