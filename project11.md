## PROJECT 11

## ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

<img width="807" alt="image" src="https://user-images.githubusercontent.com/10085348/183087436-cb5f4a74-a932-430b-b556-24c639ae96bc.png">

<img width="602" alt="image" src="https://user-images.githubusercontent.com/10085348/183087513-d2a2b216-f2bb-4b83-8f7d-99db9bcc42c0.png">

### INSTALL AND CONFIGURE ANSIBLE ON AN EC2 INSTANCE

Update Name tag on your Jenkins EC2 Instance (Used in project 9 ) to Jenkins-Ansible. We will use this server to run playbooks.

<img width="1247" alt="image" src="https://user-images.githubusercontent.com/10085348/183088949-2be6e9d8-a109-4543-9e21-d29e5527ad2f.png">

In your GitHub account create a new repository and name it ``ansible-config-mgt``


<img width="1030" alt="image" src="https://user-images.githubusercontent.com/10085348/183089715-edb06c97-2f9b-44d9-9c0c-1f95006f19ce.png">


Then install Ansible

```
sudo apt update

sudo apt install ansible
```

Check your Ansible version by running ``ansible --version``

<img width="878" alt="image" src="https://user-images.githubusercontent.com/10085348/183091022-c16c7f59-64ae-4643-b66c-bef7bc20a5f8.png">

Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

<img width="1131" alt="image" src="https://user-images.githubusercontent.com/10085348/183130320-0b4da62f-ad44-4167-8e93-07c7d779a8d0.png">

Configure Webhook in GitHub and set webhook to trigger ansible build

<img width="785" alt="image" src="https://user-images.githubusercontent.com/10085348/183130567-e3d1438b-35f8-4b98-86c2-8e300bd005b7.png">


Configure a Post-build job to save all ``(**)`` files, like you did it in Project 9

<img width="1110" alt="image" src="https://user-images.githubusercontent.com/10085348/183130673-ae9ed0c1-f704-4dff-9dca-5424e0662ea3.png">


Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```

<img width="596" alt="image" src="https://user-images.githubusercontent.com/10085348/183217985-c25b545f-207b-48cd-8e23-243bb16ab3e7.png">

<img width="973" alt="image" src="https://user-images.githubusercontent.com/10085348/183218033-c1d35523-cb71-47d6-bbb8-a0f19fef438f.png">


Every time you stop/start your Jenkins-Ansible server – you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, allocate an Elastic IP to your Jenkins-Ansible server EC2 instance


<img width="1057" alt="image" src="https://user-images.githubusercontent.com/10085348/183218244-8926dd41-d14d-4015-b9c5-9243e7c52bc7.png">

Update Github webhook payload URL

<img width="674" alt="image" src="https://user-images.githubusercontent.com/10085348/183240299-a8f6ae7e-56d1-4113-90fc-f54ad877b903.png">

Clone down your ``ansible-config-mgt`` repo to your Jenkins-Ansible instance

```
git clone <ansible-config-mgt repo link>
```

<img width="735" alt="image" src="https://user-images.githubusercontent.com/10085348/183218713-f84003cb-9f5d-4ded-9c63-25ce3667383b.png">

### BEGIN ANSIBLE DEVELOPMENT

Change to ``ansible-config-mgt`` directory

<img width="378" alt="image" src="https://user-images.githubusercontent.com/10085348/183219891-1a97d184-54cc-4779-8cbb-6a961c829c30.png">

In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

<img width="480" alt="image" src="https://user-images.githubusercontent.com/10085348/183219577-41b9afce-a3d9-4580-a7db-a76d082f6f65.png">

Renaming branch to ``development``

<img width="568" alt="image" src="https://user-images.githubusercontent.com/10085348/183220193-933480dc-551c-431f-b937-4b277ef0f2bb.png">


Create a directory and name it playbooks – it will be used to store all your playbook files.

<img width="537" alt="image" src="https://user-images.githubusercontent.com/10085348/183220232-117d71bd-1ed5-4a5b-98f7-a04da70821ad.png">


Create a directory and name it inventory – it will be used to keep your hosts organised.

<img width="476" alt="image" src="https://user-images.githubusercontent.com/10085348/183220277-1042d562-85b4-49d7-a968-3974ca0a43c9.png">


Within the playbooks folder, create your first playbook, and name it ``common.yml``

<img width="543" alt="image" src="https://user-images.githubusercontent.com/10085348/183220331-e07f7c3e-76f7-42f0-9f26-fdf3dd0ced2c.png">


Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

<img width="524" alt="Screenshot 2022-08-05 at 23 41 34" src="https://user-images.githubusercontent.com/10085348/183220577-63e1d646-c51d-48bc-9640-c2c5a077b69d.png">


setup SSH agent and connect VS Code to your Jenkins-Ansible instance

<img width="679" alt="image" src="https://user-images.githubusercontent.com/10085348/183237600-42045e17-06b9-4fd4-b235-b05c584c5264.png">

Test connection to your Jenkins-Ansible instance

<img width="639" alt="image" src="https://user-images.githubusercontent.com/10085348/183237768-ec174f29-23dc-4b41-8a65-8dcc137f67c3.png">

<img width="1025" alt="image" src="https://user-images.githubusercontent.com/10085348/183237678-4802e524-ccf9-4c4d-98d0-f0ba9d61c383.png">

Connection is successful as shown above

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

Confirm the key has been added with the command below, you should see the name of your key
```
ssh-add -l
```
<img width="707" alt="image" src="https://user-images.githubusercontent.com/10085348/183238191-07f92eb9-9ff5-4a2f-ac09-3262c23ec518.png">


Now, ssh into your Jenkins-Ansible server using ssh-agent

```
ssh -A ubuntu@public-ip
```
<img width="618" alt="image" src="https://user-images.githubusercontent.com/10085348/183238267-b479ed78-8f8e-4bc1-82bd-77e366e081ef.png">


Also notice, that your Load Balancer user and DB user is ``ubuntu`` and user for RHEL-based servers is ``ec2-user``

Update your `inventory/dev.yml` file with private IP addresses of ``NFS Server, Web Server 1 & 2, Database Server, Ngnix Load Balancer``

<img width="723" alt="image" src="https://user-images.githubusercontent.com/10085348/183241555-34ac43cf-0410-454d-afc5-c170c38dd3c7.png">


### Create a Common Playbook

Update your `playbooks/common.yml` file with following code

<img width="620" alt="image" src="https://user-images.githubusercontent.com/10085348/183238949-0119c91e-ea7e-4010-b59a-94f8e65b22a7.png">

The playbook above is divided into two parts, each of them is intended to perform the same task: 

Install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. 

It uses root user to perform this task and respective package manager: `yum` for RHEL 8 and `apt` for Ubuntu.


### Update GIT with the latest code


<img width="752" alt="image" src="https://user-images.githubusercontent.com/10085348/183239310-f7955fe4-050d-495f-ac2c-743b91bbd815.png">

<img width="588" alt="image" src="https://user-images.githubusercontent.com/10085348/183239499-5aaa3e4e-11fc-47bd-a60f-6d71889f2cf1.png">

### Create a Pull request (PR)

<img width="1010" alt="image" src="https://user-images.githubusercontent.com/10085348/183239756-60341d98-17be-4cd4-860b-eea2ec3d293c.png">

If the reviewer is happy with your new feature development, merge the code to the master branch

<img width="722" alt="image" src="https://user-images.githubusercontent.com/10085348/183239822-9daa5525-4bb9-4485-a3b3-43afb3c073a3.png">

Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes

<img width="519" alt="image" src="https://user-images.githubusercontent.com/10085348/183240075-9b8e429f-4984-4275-a21b-f93d0b2e76ec.png">

Once your code changes appear in 'master branch' – Jenkins will do its job and save all the files (build artifacts) to '/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/' directory on Jenkins-Ansible server

<img width="711" alt="image" src="https://user-images.githubusercontent.com/10085348/183240550-4ffeb532-f25d-4293-8ba4-c44737231ab3.png">

Jenkins build successful

<img width="1451" alt="image" src="https://user-images.githubusercontent.com/10085348/183240519-a816d07b-ada3-4a9a-9241-0b7dc5c5011c.png">

### Run first Ansible test

Now, it is time to execute `ansible-playbook` command and verify if your playbook actually works:

```
cd ansible-config-mgt
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```

<img width="891" alt="image" src="https://user-images.githubusercontent.com/10085348/183241156-323d4238-7f75-4267-8926-01e7068a644d.png">


You can go to each of the servers and check if wireshark has been installed by running ``which wireshark`` or ``wireshark --version``

DB Server

<img width="772" alt="image" src="https://user-images.githubusercontent.com/10085348/183241639-34b54282-3013-4de7-8fcd-8435717b9bf5.png">


NFS Server

<img width="742" alt="image" src="https://user-images.githubusercontent.com/10085348/183241709-bbbc8317-f721-478b-8776-df5f2ba06fdd.png">


Load Balancer

<img width="797" alt="image" src="https://user-images.githubusercontent.com/10085348/183241767-35e28976-8c1c-4ada-ba11-b4bd667e1f32.png">


Web Server 1

<img width="725" alt="image" src="https://user-images.githubusercontent.com/10085348/183241823-4e05f248-f7f8-44d6-93c9-69c97b91599f.png">


Web Server 2

<img width="717" alt="image" src="https://user-images.githubusercontent.com/10085348/183241871-26795ad4-429b-4b29-9ed8-03d63dc02129.png">


<img width="735" alt="image" src="https://user-images.githubusercontent.com/10085348/183241908-7b437908-c2a3-4282-adc9-08357e34eb6b.png">
