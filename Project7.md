**DEVOPS TOOLING WEBSITE SOLUTION**

<img width="793" alt="image" src="https://user-images.githubusercontent.com/10085348/176137309-8103d4d0-d1a7-488e-9bcd-a26629ea2a89.png">

Prepare NFS Server

<img width="1302" alt="Spin up a EC2 Instance with RHEL Linux 8 OS" src="https://user-images.githubusercontent.com/10085348/176137938-7e8a92a5-0d34-4b38-bdb7-8bbbea98ff8c.png">

<img width="655" alt="Configure 3 Logical Volumes lv-opt lv-apps lv-logs" src="https://user-images.githubusercontent.com/10085348/176138341-7b5cff55-82aa-4a45-9030-b173c74eaaf1.png">

Format the disks as xfs

<img width="482" alt="Configure access to NFS for clients within the same subnet" src="https://user-images.githubusercontent.com/10085348/176142278-a9874a10-6ce9-4950-9e71-bc6ebb1ad20e.png">


<img width="738" alt="Format disks as xfs" src="https://user-images.githubusercontent.com/10085348/176138637-f30086cc-6fe5-49f2-8830-ed3299518be5.png">


Create directories for apps, opt, logs<img width="1220" alt="AWS Inbound rules" src="https://user-images.githubusercontent.com/10085348/176142910-4e2f639a-2425-4c53-b356-2a01e975fa95.png">


<img width="377" alt="Create directories for logical volume mount points" src="https://user-images.githubusercontent.com/10085348/176139646-2a449a01-0683-4fd7-aca8-44fae6fe3b26.png">

Create mount points on /mnt directory for the logical volumes

<img width="555" alt="Mount lv-app, lv-logs, lv-opt " src="https://user-images.githubusercontent.com/10085348/176140527-4f7ec2fe-6b1c-49fd-8917-63d43ddd011a.png">



Install NFS server
 
Use the following commands to install NFS Server:
 
sudo yum -y update

sudo yum install nfs-utils -y

sudo systemctl start nfs-server.service

sudo systemctl enable nfs-server.service

sudo systemctl status nfs-server.service

<img width="935" alt="Install NFS Server" src="https://user-images.githubusercontent.com/10085348/176139172-389cd9bd-e5ba-4e0b-9a66-c819bfe688a7.png">


Configure access to NFS for clients within the same subnet(172.31.32.0/20)

<img width="482" alt="Configure access to NFS for clients within the same subnet" src="https://user-images.githubusercontent.com/10085348/176142385-78aaea77-c2f8-48fa-883f-94e7cba5550c.png">

Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

<img width="481" alt="check which ports used by NFS" src="https://user-images.githubusercontent.com/10085348/176142765-c4287390-2909-4cc6-af2e-9ba164338bf7.png">

<img width="1220" alt="AWS Inbound rules" src="https://user-images.githubusercontent.com/10085348/176142975-8660a40c-7975-414b-a5f1-63cc4665a119.png">


**CONFIGURE THE DATABASE SERVER**

Install MySQL Server

<img width="1013" alt="Install mysql server" src="https://user-images.githubusercontent.com/10085348/176148077-6957bffd-ba6e-4c7f-abd5-70fd01df21f7.png">

<img width="801" alt="Check my mysql server status" src="https://user-images.githubusercontent.com/10085348/176148857-405bfb18-a43c-430e-8e22-5dc2c95117b7.png">

Create a database "tooling", add User "webaccess" and grant privileges

<img width="803" alt="Create Tooling DB plus add user webaccess and grant privileges" src="https://user-images.githubusercontent.com/10085348/176148258-32894ff7-1af5-4e7b-ac91-7d546f85212a.png">

<img width="839" alt="Edit mysql bind address" src="https://user-images.githubusercontent.com/10085348/176148941-985058dd-a980-468a-b842-0ea1a3a8e0e2.png">


**Prepare the Web Servers**

<img width="1214" alt="image" src="https://user-images.githubusercontent.com/10085348/176150171-0312398c-66f3-4390-b858-1920a17dc634.png">

Install NFS client

<img width="765" alt="image" src="https://user-images.githubusercontent.com/10085348/176151660-3b77de0e-ad51-497b-a977-b4534cb1787e.png">

Mount /var/www/ and target the NFS server’s export for apps
Use the following commands:

sudo mkdir /var/www


<img width="671" alt="image" src="https://user-images.githubusercontent.com/10085348/176153811-2e2ab1d9-bb06-488e-9982-417a0577b590.png">
       
my NFS Server Private ip is 172.31.41.194


 Verify that NFS was mounted successfully by running df -h
  
  <img width="502" alt="image" src="https://user-images.githubusercontent.com/10085348/176152344-0440b8d1-ea75-44d1-b9da-d232ca4383ee.png">

  Make sure that the changes will persist on Web Server after reboot
  
  run the command: sudo vi /etc/fstab
  
  and add following line
  
 <img width="636" alt="image" src="https://user-images.githubusercontent.com/10085348/176153871-65d4eabe-81d6-4f29-8fbc-e9d67f4ac1d8.png">

<img width="678" alt="image" src="https://user-images.githubusercontent.com/10085348/176152837-f90ddff9-4871-4a8f-9959-29968534376e.png">

Install Remi’s repository, Apache and PHP

sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1

Repeat steps 1-5 for Web Servers 2 and 3

Install git

sudo yum install git -y

Run command to Fork the tooling source code from https://github.com/darey-io/tooling.git

git clone https://github.com/darey-io/tooling.git

Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html

<img width="980" alt="image" src="https://user-images.githubusercontent.com/10085348/176155949-57299f7b-abc3-41d7-ae36-20e2708a117b.png">


Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). 

<img width="720" alt="image" src="https://user-images.githubusercontent.com/10085348/176156589-a5de90c3-10a0-44ec-b0a0-c0fce5f753e1.png">

Apply tooling-db.sql script to your database 

<img width="791" alt="image" src="https://user-images.githubusercontent.com/10085348/176156914-4048fde1-2aa5-463f-b335-61b3dbe1524b.png">

Verify you can access Database Server from WebServer

<img width="752" alt="image" src="https://user-images.githubusercontent.com/10085348/176157934-d82fd6ce-603d-4f66-aa11-7696d88f5684.png">


Open the website in your browser 

Type in:

http://Web-Server-Public-IP-Address/index.php 

<img width="1230" alt="Propitix login" src="https://user-images.githubusercontent.com/10085348/176157091-16474ec3-b0ac-42a0-9e34-223f7dcfb96b.png">

<img width="1492" alt="Admin login" src="https://user-images.githubusercontent.com/10085348/176157639-9369b5f5-6382-44ee-bb42-f475e3603279.png">


