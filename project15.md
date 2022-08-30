## Project 15

### AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

### Starting Off Your AWS Cloud Project

Configure your AWS Master account and Organization Unit


<img width="1141" alt="Screenshot 2022-08-30 at 12 11 33" src="https://user-images.githubusercontent.com/10085348/187422936-4e216b24-e269-45c9-8f04-287d802db744.png">


Within the Root account, create a sub-account and name it **DevOps**. (You will need another email address to complete this)

Within the Root account, create an **AWS Organization Unit (OU)**. Name it **Dev**. (We will launch Dev resources in there)

Move the DevOps account into the **Dev OU**.


![image](https://user-images.githubusercontent.com/10085348/187423809-cd494c95-1bde-4265-a973-416cc206688b.png)

Login to the newly created account

<img width="1511" alt="Screenshot 2022-08-30 at 12 22 46" src="https://user-images.githubusercontent.com/10085348/187424542-60c0a41a-4cea-46d0-aa6d-6ce9bb220033.png">

Created a domain name from GoDaddy

Create a hosted zone in AWS Route 53

<img width="1028" alt="image" src="https://user-images.githubusercontent.com/10085348/187428684-a5dacfb1-bfc5-4b84-8f9b-3d792a81a25f.png">

Click on the created hosted zone and copy the contents of the NS record

<img width="1103" alt="image" src="https://user-images.githubusercontent.com/10085348/187429654-6a561723-cf22-432d-9186-b7332c36dbfd.png">


Replace the content there with the items you got from Hosted zone to GoDaddy's custom nameservers

<img width="1226" alt="image" src="https://user-images.githubusercontent.com/10085348/187430272-03cbe5bf-526d-4976-be44-3664e586f79b.png">


NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:
```
Project: <Give your project a name>
Environment: <dev>
Automated: <No> (If you create a recource using an automation tool, it would be <Yes>)
```

## Setup a Virtual Private Cloud

![image](https://user-images.githubusercontent.com/10085348/187431059-466a003f-6652-499e-83ff-dc323252c88a.png)


Always make reference to the architectural diagram and ensure that your configuration is aligned with it.

- Create a VPC

- Create subnets as shown in the architecture

- Create a route table and associate it with public subnets

- Create a route table and associate it with private subnets

- Create an Internet Gateway

- Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

- Create 3 Elastic IPs

- Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

- Create a Security Group for:

**Nginx Servers:** Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.

**Bastion Servers:** Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

**Application Load Balancer:** ALB will be available from the Internet

**Webservers:** Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.

**Data Layer:** Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.


