**Project 5**

**Implementing a Client Server Architecture using MySQL Database Management System (DBMS)**

Create and configure two Linux-based virtual servers (EC2 instances in AWS)

<img width="1512" alt="Step up mysql server and client on AWS" src="https://user-images.githubusercontent.com/10085348/170649721-55f9ca86-a341-42d3-9616-9b5d6045ea33.png">

Server A name - `mysql server`

<img width="1175" alt="Installing mysql server" src="https://user-images.githubusercontent.com/10085348/170652077-2e7373d3-cc4a-4add-9186-700e1c0bc45c.png">


Run a security script that comes pre-installed with MySQL to remove some insecure default settings and lock down access to your database system

<img width="871" alt="Run a security script that comes pre-installed with MySQL to remove some insecure default settings and lock down access to your database system" src="https://user-images.githubusercontent.com/10085348/170651456-2d437b76-5811-49f4-bce2-b1ed97527793.png">

Server B name - `mysql client`

<img width="1096" alt="Mysql client Installation" src="https://user-images.githubusercontent.com/10085348/170650018-f9d5566e-a0b2-48f7-ad90-efc2a89a26d0.png">

configure MySQL server to allow connections from remote hosts

<img width="947" alt="mysql bind address " src="https://user-images.githubusercontent.com/10085348/170650311-f1174c58-c9ea-495a-8ba7-5678353cc46f.png">

 MySQL server uses TCP port 3306 by default, open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ server security group on AWS EC2 instance

<img width="1481" alt="mysql Inbound rules" src="https://user-images.githubusercontent.com/10085348/170650525-788a924e-54d1-474a-bda5-535b63fce9e1.png">

Create a Firewall rule to allow remote connection from MySQL client to Server

<img width="515" alt="Firewall rule to allow remote connection" src="https://user-images.githubusercontent.com/10085348/170652889-f519b49f-4bd6-4d11-a9a2-7bc0e85aa047.png">

MySQL client Linux Server remote connection to MySQL server Database Engine

<img width="1012" alt="Client connection to mysqlserver" src="https://user-images.githubusercontent.com/10085348/170652586-42f1a3d4-6bf3-48a1-a33b-ec45813889c3.png">

Check that you have successfully connected to a remote MySQL server and can perform SQL queries
The command (show databases;) highlighted in the picture above can be used to show current databases on the mySql Server
