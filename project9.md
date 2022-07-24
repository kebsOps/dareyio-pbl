## PROJECT 9

## INSTALL AND CONFIGURE JENKINS SERVER

<img width="909" alt="image" src="https://user-images.githubusercontent.com/10085348/180647137-dd988d63-e8e3-49b6-9f69-dc9d2cd8b888.png">


### Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

### Install JDK (since Jenkins is a Java-based application)


```
sudo apt update
```
```
sudo apt install default-jdk-headless

```

<img width="1105" alt="apt update and openjdk installation" src="https://user-images.githubusercontent.com/10085348/180647330-bb99dff3-7344-4a74-a77c-d6d1fa39f035.png">


### Install Jenkins

<img width="823" alt="Jenkins Installation" src="https://user-images.githubusercontent.com/10085348/180647446-ad76b4b0-5823-48f5-990e-1a98746758a4.png">

### Verify that Jenkins is up and running
<img width="1188" alt="Jenkins is up and running" src="https://user-images.githubusercontent.com/10085348/180647473-130c4d89-87d4-4bb9-a618-4040347f97f8.png">

### By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

<img width="785" alt="Security group 8080" src="https://user-images.githubusercontent.com/10085348/180647560-07ff3471-5b10-4230-ade3-9e17e61f8750.png">


### Perform initial Jenkins setup.


From your browser access
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```
You will be prompted to provide a default admin password


<img width="1285" alt="Unlock Jenkins" src="https://user-images.githubusercontent.com/10085348/180648236-a0b73f95-9533-41c9-ac3f-ab022160c172.png">





