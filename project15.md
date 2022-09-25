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

Created a domain name from GoDaddy `toolingkb.xyz`

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


**3 EC2 Instance based on Red Hat of the t2.micro family were launched for nginx, bastion and the one for the two webservers**

### Set Up Compute Resources for Bastion 

Ensure that it has the following software installed
- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop

```
sudo su -

yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd 

systemctl enable chronyd
```
<img width="1178" alt="image" src="https://user-images.githubusercontent.com/10085348/187676610-2026a629-66e7-424f-bf0c-14ac1428478d.png">



### Set Up Compute Resources for Nginx

Ensure that it has the following software installed:
- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop

```
sudo su -

yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

<img width="1223" alt="image" src="https://user-images.githubusercontent.com/10085348/187678248-39f91499-b187-4c68-90bd-b57fd03bab6a.png">

Configure selinux policies for Nginx servers

```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```

Install amazon efs utils for mounting the target on the Elastic file system

```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```

Seting up self-signed certificate for the nginx instance
```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/Kebs.key -out /etc/ssl/certs/Kebs.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
<img width="1198" alt="Screenshot 2022-08-31 at 13 46 30" src="https://user-images.githubusercontent.com/10085348/187681635-aafa6638-e935-4c42-bd2c-a8d5c7423b2b.png">

To verify

<img width="1189" alt="image" src="https://user-images.githubusercontent.com/10085348/187681734-ea8d4adf-da4c-4eaf-b5f4-1b4495eb58ff.png">

<img width="417" alt="image" src="https://user-images.githubusercontent.com/10085348/187681920-8950ec77-06e5-4959-abab-d3f7615c444c.png">


**Set Up Compute Resources for Websevers**

```
sudo su -

yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```
<img width="1225" alt="image" src="https://user-images.githubusercontent.com/10085348/187683388-f019dd9b-a3a1-45de-b67b-52e34d20f30b.png">


Configure selinux policies for web servers

```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1

```

Install amazon efs utils for mounting the target on the Elastic file system

```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm

```

Setting up self-signed certificate for the apache webserver instance
```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/Kebs.key -x509 -days 365 -out /etc/pki/tls/certs/Kebs.crt

vi /etc/httpd/conf.d/ssl.conf

```

<img width="591" alt="image" src="https://user-images.githubusercontent.com/10085348/187687554-2f202e4e-9dbf-4772-a29d-ca7fd1f78945.png">


## Create an AMI out of the EC2 instances

<img width="1308" alt="image" src="https://user-images.githubusercontent.com/10085348/187692249-5b4094ce-e1fc-4642-8eda-15f9baad076a.png">



## Configure Target Groups

<img width="1251" alt="image" src="https://user-images.githubusercontent.com/10085348/187691563-2d0244c5-f919-4a4f-80e8-9807dabd6a66.png">


### Configure Applipcation Load balancers (ALB) (one External, One Internal)

Application Load Balancer To Route Traffic To NGINX


Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

```
Create an Internet facing ALB
Ensure that it listens on HTTPS protocol (TCP port 443)
Ensure the ALB is created within the appropriate VPC | AZ | Subnets
Choose the Certificate from ACM
Select Security Group
Select Nginx Instances as the target group
```

<img width="1304" alt="image" src="https://user-images.githubusercontent.com/10085348/187908748-812af219-bce3-4ec6-ad60-568b5387071e.png">

<img width="1279" alt="image" src="https://user-images.githubusercontent.com/10085348/187908828-7f8cfa78-fb92-48b5-9d6a-2f3ed0eb0841.png">


<img width="1319" alt="image" src="https://user-images.githubusercontent.com/10085348/187695151-2e488179-600d-466b-9b6c-a8edb851cae3.png">


On the Internal ALB

```
Ensure that it listens on HTTPS protocol (TCP port 443)

Ensure the ALB is created within the appropriate VPC | AZ | Subnets

Choose the Certificate from ACM

Select Security Group

Select webserver Instances as the target group

Ensure that health check passes for the target group
```

<img width="1312" alt="image" src="https://user-images.githubusercontent.com/10085348/187909178-ad933bf6-58b6-406e-8733-550586ef6f26.png">


<img width="1039" alt="image" src="https://user-images.githubusercontent.com/10085348/187695872-f6649afe-49b4-4008-99a0-5d01042d6a3d.png">




## Create Launch Templates

Bastion Launch Template

<img width="654" alt="image" src="https://user-images.githubusercontent.com/10085348/187699186-8829f708-c507-48f8-ae60-7e7515b1a0f4.png">

<img width="951" alt="image" src="https://user-images.githubusercontent.com/10085348/187699379-d4bbd4ed-5b5d-4be4-8919-9096a231c7f2.png">


<img width="629" alt="image" src="https://user-images.githubusercontent.com/10085348/187698927-1a4d7419-fe41-4f1a-9e73-f272f3799a3a.png">


Bastion Launch Template User Data

```
#!/bin/bash 
yum install -y mysql 
yum install -y git tmux 
yum install -y ansible

```
<img width="599" alt="image" src="https://user-images.githubusercontent.com/10085348/187698791-a77b2770-c97d-4393-a51d-a472dbf95fce.png">


Ngnix Launch Template


<img width="655" alt="image" src="https://user-images.githubusercontent.com/10085348/187706158-dd0e122a-f8e6-4906-b6db-ba2f5127c22e.png">

<img width="642" alt="image" src="https://user-images.githubusercontent.com/10085348/187706272-0c4bcd19-b62a-400f-a9f1-c3a5ceec1efa.png">

<img width="623" alt="image" src="https://user-images.githubusercontent.com/10085348/187706692-e9771b8c-e75e-4b22-a54b-1ce3e3324a96.png">


Ngnix Launch Template User Data

<img width="581" alt="image" src="https://user-images.githubusercontent.com/10085348/187706059-e4a54c08-ba0d-4576-a19a-21b52bf84f9a.png">

```
#!/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/kebsOps/ACS-project-config.git
mv /ACS-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /ACS-project-config
```

Wordpress Launch Template

<img width="660" alt="image" src="https://user-images.githubusercontent.com/10085348/187716980-c4ece9ec-fef4-4790-aed6-8883a8bbef83.png">

<img width="639" alt="image" src="https://user-images.githubusercontent.com/10085348/187717127-b60a6204-f02e-4785-8351-d253c2673ce4.png">

<img width="620" alt="image" src="https://user-images.githubusercontent.com/10085348/187717325-182a55f4-fd71-4515-8824-8aad9f4cb58e.png">


Wordpress Launch Template User Data

```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-08334da6d7b10fdec fs-0a1913a76c4e8545b:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/database-1.cy2s6kbdev0j.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/Kebsadmin/g" wp-config.php 
sed -i "s/password_here/admin12345/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

```

<img width="969" alt="image" src="https://user-images.githubusercontent.com/10085348/187717486-323b933a-5afd-455e-8dbd-bb1e77a7677e.png">

Tooling launch Template

<img width="631" alt="image" src="https://user-images.githubusercontent.com/10085348/187721056-664c0b57-27d0-423c-9c50-915b2eb11ae5.png">

<img width="637" alt="image" src="https://user-images.githubusercontent.com/10085348/187721132-4a17ed52-cd44-4478-bbc2-0d32eaac775a.png">

<img width="616" alt="image" src="https://user-images.githubusercontent.com/10085348/187721355-4fac6ad5-5c71-4b41-92cb-ebeb56e78e41.png">


Tooling launch Template User Data
```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-038e3c726cbc66b11 fs-0a1913a76c4e8545b:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/kebsOps/tooling.git
mkdir /var/www/html
cp -R /tooling-1/html/*  /var/www/html/
cd /tooling-1
mysql -h database-1.cy2s6kbdev0j.us-east-1.rds.amazonaws.com -u ACSadmin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('database-1.cy2s6kbdev0j.us-east-1.rds.amazonaws.com', 'Kebsadmin', 'admin12345', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
```

<img width="1020" alt="image" src="https://user-images.githubusercontent.com/10085348/187721477-6bf7f6b0-dbf6-4e95-809b-5d2f7995061d.png">


<img width="1252" alt="image" src="https://user-images.githubusercontent.com/10085348/187721706-82b3c419-29fc-4641-9860-be902b31c864.png">

[Link to Project user data config scripts Repo](https://github.com/kebsOps/ACS-project-config)


## Configure Auto Scaling Groups

Configure Autoscaling group For Bastion

<img width="1281" alt="image" src="https://user-images.githubusercontent.com/10085348/187905571-36cc05e2-ebd1-46ab-ada8-1d266310133a.png">

<img width="1269" alt="image" src="https://user-images.githubusercontent.com/10085348/187905670-0313821f-a40d-4a50-9263-46356cd30d2a.png">



Configure Autoscaling Group For Ngnix

<img width="1264" alt="image" src="https://user-images.githubusercontent.com/10085348/187905293-b6a8cac5-3afa-4c52-9028-e66b6272895f.png">

<img width="1267" alt="image" src="https://user-images.githubusercontent.com/10085348/187905403-f5e190dd-bfbe-44b2-a3ab-34ef00508a78.png">


Configure Autoscaling Group For Wordpress

Prerequisite

- Create `wordpressdb` and `toolingdb` on Kebs-Bastion launch template

<img width="770" alt="image" src="https://user-images.githubusercontent.com/10085348/187871576-5b549d58-5472-4e09-b7e5-4a394e10e4a8.png">

<img width="692" alt="image" src="https://user-images.githubusercontent.com/10085348/187886301-00f36672-29e8-406b-b0cb-7b1d32e7d330.png">

<img width="654" alt="image" src="https://user-images.githubusercontent.com/10085348/187886359-699950a6-7a4a-4a35-a148-16767d035fcd.png">

<img width="634" alt="image" src="https://user-images.githubusercontent.com/10085348/187886443-0aabcdbf-d20c-4984-9338-5d2882306157.png">

Configure Autoscaling Group For tooling

<img width="633" alt="image" src="https://user-images.githubusercontent.com/10085348/187887442-4b7029cb-00c0-482e-9745-5a06ed6934d6.png">

<img width="689" alt="image" src="https://user-images.githubusercontent.com/10085348/187887666-8494b729-4343-4d3a-90cd-8834dc2089db.png">

<img width="630" alt="image" src="https://user-images.githubusercontent.com/10085348/187887748-2fa3ab2f-af3b-434f-8f04-21763de62e6d.png">



<img width="1253" alt="image" src="https://user-images.githubusercontent.com/10085348/187905058-53ba2e39-7192-4075-818b-ea9eb938e177.png">


Checks on wordpress server

<img width="577" alt="image" src="https://user-images.githubusercontent.com/10085348/187894094-42247e1b-72a3-4bc1-aee5-ead02233eaaa.png">

<img width="572" alt="image" src="https://user-images.githubusercontent.com/10085348/187894221-833ad149-49e6-49c8-8802-943a3b903eb9.png">

<img width="899" alt="image" src="https://user-images.githubusercontent.com/10085348/187894430-dffcc842-5de4-454e-a5d6-f2681093c2dc.png">


TLS Certificates from Amazon Certificate Manager (ACM)

You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

Navigate to AWS ACM
Request a public wildcard certificate for the domain name you registered in ``GoDaddy.com``
Use DNS to validate the domain name
Tag the resource

<img width="1413" alt="image" src="https://user-images.githubusercontent.com/10085348/187671849-8a683a45-b05c-4a48-affb-9caed31a3d60.png">

<img width="1411" alt="image" src="https://user-images.githubusercontent.com/10085348/187906011-134bd1f1-b846-4a21-8ca9-9abbbd6fa2d6.png">

<img width="1389" alt="image" src="https://user-images.githubusercontent.com/10085348/187906112-fba93a2d-8b02-44d9-bdfa-ac44fb2d4d7c.png">



Setup EFS

<img width="1270" alt="image" src="https://user-images.githubusercontent.com/10085348/187650917-c9eecb59-5276-4ffa-8879-c98d4c734123.png">

Setup Access Points

wordpress AP
<img width="1282" alt="image" src="https://user-images.githubusercontent.com/10085348/187652174-0777b531-8218-47eb-b8f6-f7cba8c56b5e.png">

tooling AP
<img width="1271" alt="image" src="https://user-images.githubusercontent.com/10085348/187652956-71cce6e3-3ec1-44cd-bd18-0c54fea633b0.png">

<img width="1265" alt="image" src="https://user-images.githubusercontent.com/10085348/187653157-a633ba5b-2450-44f9-be3f-632b3a0e7089.png">


Setup RDS

Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

<img width="1463" alt="image" src="https://user-images.githubusercontent.com/10085348/187654697-f78806de-c4bb-4698-82ce-749ebe9b9c63.png">

Create a subnet group and add 2 private subnets (data Layer)

<img width="1294" alt="Screenshot 2022-08-31 at 11 13 28" src="https://user-images.githubusercontent.com/10085348/187655609-cba03985-25a0-4f72-b7ed-0eca3eae1795.png">


Creating DNS Records In The Route53 For the Tooling And Wordpress site

<img width="804" alt="image" src="https://user-images.githubusercontent.com/10085348/187892506-d2804443-35db-4317-8cb2-0cb70c38364a.png">

<img width="800" alt="image" src="https://user-images.githubusercontent.com/10085348/187892657-6e9fe9ea-c200-407a-8e1c-ec6d2d2225b1.png">

<img width="818" alt="image" src="https://user-images.githubusercontent.com/10085348/187892879-7e4a7ba8-0c90-4a75-bc61-4f2ec9fe76cd.png">

<img width="902" alt="image" src="https://user-images.githubusercontent.com/10085348/187893159-8986d287-183c-4f47-b337-0d0682a51fb4.png">



WordPress Site


<img width="1476" alt="Screenshot 2022-09-01 at 12 09 53" src="https://user-images.githubusercontent.com/10085348/187900364-80f3bc8b-c032-4c34-9b0a-d46eca595cd7.png">


Tooling Site

<img width="1483" alt="Screenshot 2022-09-01 at 12 17 48" src="https://user-images.githubusercontent.com/10085348/187901859-98e45787-797e-434e-b0f5-80d8e27c7177.png">


[Link to Project config scripts Repo](https://github.com/kebsOps/ACS-project-config)

