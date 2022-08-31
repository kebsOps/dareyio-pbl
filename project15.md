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

<img width="1218" alt="image" src="https://user-images.githubusercontent.com/10085348/187493042-e330a1c0-0ef1-4832-8c41-5634b662cc08.png">


- Create subnets as shown in the architecture

<img width="1244" alt="image" src="https://user-images.githubusercontent.com/10085348/187494624-2ab88507-3547-4bc3-80dc-844d8395b790.png">


- Create two route tables and associate it with respective subnets (public and private)

<img width="1257" alt="image" src="https://user-images.githubusercontent.com/10085348/187496194-017d9a61-e0af-436c-857c-4079ee2304e7.png">

- Create an Internet Gateway

<img width="1047" alt="image" src="https://user-images.githubusercontent.com/10085348/187497128-027382fd-54e8-44a7-9577-e9cdc7249ee7.png">

<img width="1222" alt="image" src="https://user-images.githubusercontent.com/10085348/187497404-08dbd149-03ef-4f35-94c3-f0c07b6ac9db.png">


- Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

<img width="1475" alt="image" src="https://user-images.githubusercontent.com/10085348/187498847-e1bdf8b9-01f8-4261-a4f0-4286acb0cc8e.png">


- Create 3 Elastic IPs

<img width="1220" alt="image" src="https://user-images.githubusercontent.com/10085348/187502353-696c0ebf-dc77-4650-81f7-63d64a35a69f.png">


- Create a NAT Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

<img width="763" alt="image" src="https://user-images.githubusercontent.com/10085348/187507246-51493990-12e0-4851-aaba-8a71b86ddba2.png">

<img width="1216" alt="image" src="https://user-images.githubusercontent.com/10085348/187510965-6fe2c2ef-934d-4da2-9517-e9ed8dfe4ae7.png">

- Create a Security Group for:

**Nginx Servers:** Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.

<img width="1290" alt="image" src="https://user-images.githubusercontent.com/10085348/187517129-a1fbc510-6952-4dbe-af64-fe86adf60265.png">


**Bastion Servers:** Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

<img width="1427" alt="image" src="https://user-images.githubusercontent.com/10085348/187515273-9344123f-2649-474d-b3ec-7bd4f2a2bfcd.png">


**Application Load Balancer:** ALB will be available from the Internet

<img width="1249" alt="image" src="https://user-images.githubusercontent.com/10085348/187513816-55097a66-48ec-4541-b56d-ec28c61b3ced.png">

<img width="1248" alt="image" src="https://user-images.githubusercontent.com/10085348/187513896-3f01856b-8a3e-49eb-876a-7c4d7bd6cac7.png">

**Internal Load Balancer**
<img width="1413" alt="image" src="https://user-images.githubusercontent.com/10085348/187519927-9c463061-5baf-40ef-afde-ced32eeac0b1.png">


**Webservers:** Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.

<img width="1271" alt="image" src="https://user-images.githubusercontent.com/10085348/187521144-15f6346b-2f78-4a66-b906-1c70316b788d.png">


**Data Layer:** Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully designed – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

<img width="1277" alt="image" src="https://user-images.githubusercontent.com/10085348/187523698-127f7ddf-a60e-456c-b94f-ce7ee7b97839.png">


<img width="1292" alt="image" src="https://user-images.githubusercontent.com/10085348/187523969-306c461e-3770-49f0-9295-5439be81600b.png">


