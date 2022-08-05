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



