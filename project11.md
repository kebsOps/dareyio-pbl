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

<img width="691" alt="image" src="https://user-images.githubusercontent.com/10085348/183218403-4f0a39fd-5739-40d9-bfc0-3e5522885299.png">

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




