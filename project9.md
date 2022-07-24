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



### Admin password gotten with the command below:

<img width="612" alt="Jenkins Initial Admin Password" src="https://user-images.githubusercontent.com/10085348/180651320-b83cab07-52fb-43ea-9585-917d48284a1d.png">



<img width="1302" alt="Install Jenkins" src="https://user-images.githubusercontent.com/10085348/180651405-ce5832d1-e05b-4528-84b6-4f4113eedde4.png">

<img width="1446" alt="Instance Configuration IP" src="https://user-images.githubusercontent.com/10085348/180651491-8db24c61-213e-4e68-8337-555ff93d8d7a.png">


<img width="1346" alt="Jenkins is ready" src="https://user-images.githubusercontent.com/10085348/180651522-b388b854-cb36-4f9b-975f-86360158fa4b.png">


### Configure Jenkins to retrieve source codes from GitHub using Webhooks

### Enable webhooks in your GitHub repository settings This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

<img width="789" alt="Webhook" src="https://user-images.githubusercontent.com/10085348/180651896-0897f3d7-84e8-48c3-bafa-b2b736a94cf9.png">

### Go to Jenkins web console, click "New Item" and create a "Freestyle project" call it tooling_github

<img width="1462" alt="Project tooling_github build 1#" src="https://user-images.githubusercontent.com/10085348/180651955-a0063ad8-dd47-49ad-9a8e-7c8644945b1a.png">


### Click "Configure" your job/project and add these two configurations

### Configure triggering the job from GitHub webhook:

<img width="449" alt="Build Triggers" src="https://user-images.githubusercontent.com/10085348/180652212-dc7d44e6-45bc-4daf-8b8c-06c22506607d.png">

### Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

<img width="1017" alt="Archive the artifacts" src="https://user-images.githubusercontent.com/10085348/180652373-151364db-befc-4e13-9018-67ca81fa8892.png">

### Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch. You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

<img width="1456" alt="Build 2" src="https://user-images.githubusercontent.com/10085348/180652423-24116b45-09b5-4248-a457-5ecf7cf44ad9.png">

### By default, the artifacts are stored on Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```

# Install Publish over SSH Plugin and complete setup


<img width="1444" alt="Publish over SSH plugin" src="https://user-images.githubusercontent.com/10085348/180652465-88a7727d-8fa2-4584-b3e1-af6361ce1b11.png">

<img width="1029" alt="SSH Servers" src="https://user-images.githubusercontent.com/10085348/180652520-10248018-7d6f-4d6e-86fc-46f6fbd593b3.png">


<img width="1000" alt="Publish over ssh private key" src="https://user-images.githubusercontent.com/10085348/180652544-988070be-93bf-4fe5-9a27-05bd00ed079b.png">


<img width="1176" alt="Send build artifacts over SSH" src="https://user-images.githubusercontent.com/10085348/180652563-fdb48d83-8f7d-4355-b78f-f19d51353ee8.png">


### Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository. Webhook will trigger a new job and in the "Console Output" of the job

<img width="1307" alt="build successful" src="https://user-images.githubusercontent.com/10085348/180652695-ea61ec24-9420-452e-83d3-1246f66472d4.png">


<img width="1474" alt="Files transferred" src="https://user-images.githubusercontent.com/10085348/180652701-79010087-24a4-4f81-9877-ad1fd19f0b58.png">

### To make sure that the files in ``` /mnt/apps ``` have been udated – connect via SSH/Putty to your NFS server and check README.MD file

```
cat /mnt/apps/README.md
```

<img width="862" alt="updated webhook" src="https://user-images.githubusercontent.com/10085348/180652708-61254769-a462-4c74-a2d2-244a031a984e.png">

### If you see the changes you had previously made in your GitHub – the job works as expected.


