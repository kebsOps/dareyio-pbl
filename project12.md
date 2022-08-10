## PROJECT 12

## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

## Code Refactoring
Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. 
The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, 
reduce complexity, add proper comments without affecting the logic.

## Jenkins job enhancement

Go to your Jenkins-Ansible server and create a new directory called ``ansible-config-artifact`` – we will store there all artifacts after each build.


`` sudo mkdir /home/ubuntu/ansible-config-artifact ``


![Screenshot 2022-08-08 at 17 02 08](https://user-images.githubusercontent.com/10085348/183463751-c3dc74ca-6fa4-47fd-88ce-efa0ca85efbe.png)


Change permissions to this directory, so Jenkins could save files there – 

``chmod -R 0777 /home/ubuntu/ansible-config-artifact``

Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> 
on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

<img width="1111" alt="image" src="https://user-images.githubusercontent.com/10085348/183465052-7b953992-f7ec-4407-8bc6-3a2ae2d0549c.png">

<img width="576" alt="image" src="https://user-images.githubusercontent.com/10085348/183465231-7a7d1bd3-a55f-4854-99b9-75a1283010b6.png">


Create a new Freestyle project (you have done it in Project 9) and name it ``save_artifacts``

<img width="950" alt="image" src="https://user-images.githubusercontent.com/10085348/183467163-eac7ad75-acc7-44aa-b1c9-e525f7a740a2.png">

This project will be triggered by completion of your existing ansible project. Configure it accordingly:

<img width="972" alt="image" src="https://user-images.githubusercontent.com/10085348/183466535-1e42f669-e8e2-47e1-bddb-629e77f5a002.png">

<img width="972" alt="image" src="https://user-images.githubusercontent.com/10085348/183466691-323e5ad9-dd61-424a-8b8e-78950426e813.png">

<img width="987" alt="image" src="https://user-images.githubusercontent.com/10085348/183469229-978efec6-8ff3-4389-ad04-c76187da1040.png">

Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).

<img width="1451" alt="image" src="https://user-images.githubusercontent.com/10085348/183472971-4758a742-fb1c-48a6-9d7e-0a808b0c202c.png">


If both Jenkins jobs have completed one after another – you shall see your files inside ``/home/ubuntu/ansible-config-artifact`` directory and it will be updated with every commit to your master branch.

<img width="1451" alt="image" src="https://user-images.githubusercontent.com/10085348/183472020-cd929845-2950-4dc6-8e64-7c6015a2e21f.png">


Now your Jenkins pipeline is more neat and clean


<img width="692" alt="image" src="https://user-images.githubusercontent.com/10085348/183472504-5c507c9b-9766-4946-8221-7730db71a22a.png">


### Refactor Ansible code by importing other playbooks into `site.yml'

Pull down the latest code from master (main) branch, and created a new branch, name it 'refactor'


<img width="453" alt="image" src="https://user-images.githubusercontent.com/10085348/183748157-3cff7ed6-deb5-4c17-8ede-251c507189d3.png">

<img width="532" alt="image" src="https://user-images.githubusercontent.com/10085348/183749898-e19caf3d-c45b-400b-a55a-f144e9ef6e69.png">


DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

### Let see code re-use in action by importing other playbooks

Within playbooks folder, create a new file and name it `site.yml` – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, `site.yml` will become a parent to all other playbooks that will be developed. Including `common.yml` that you created previously.

<img width="526" alt="image" src="https://user-images.githubusercontent.com/10085348/183748808-48fe2713-dbf6-45c2-8045-4233157a6bd0.png">

Create a new folder in root of the repository and name it `static-assignments`. The `static-assignments folder` is where all other children playbooks will be stored. This is merely for easy organization of your work. 

<img width="518" alt="image" src="https://user-images.githubusercontent.com/10085348/183751948-f2e9f3de-04d8-4b0a-a79f-a92021f81e83.png">

Move `common.yml` file into the newly created `static-assignments` folder

<img width="926" alt="image" src="https://user-images.githubusercontent.com/10085348/183752507-ddf0d478-6684-4569-948c-e320c1c60520.png">

Inside `site.yml` file, import `common.yml` playbook

<img width="501" alt="image" src="https://user-images.githubusercontent.com/10085348/183753225-d2b9c631-128d-49af-a586-fcbff939c560.png">


The code above uses built in import_playbook Ansible module.

<img width="897" alt="image" src="https://user-images.githubusercontent.com/10085348/183753493-2e7a2723-1afc-47aa-af84-f125d41c3ec0.png">


Run `ansible-playbook` command against the `dev` environment

Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under `static-assignments` and name `it common-del.yml`. 

<img width="626" alt="image" src="https://user-images.githubusercontent.com/10085348/183755374-6842f3ad-0000-47b5-b8c7-ab57c45dde55.png">


In this playbook, configure deletion of wireshark utility.

<img width="605" alt="image" src="https://user-images.githubusercontent.com/10085348/183755925-6ee0a644-ed96-47ac-8d2f-6de76a327635.png">


update `site.yml` with `- import_playbook: ../static-assignments/common-del.yml` instead of `common.yml` and run it against dev servers:

<img width="626" alt="image" src="https://user-images.githubusercontent.com/10085348/183755374-6842f3ad-0000-47b5-b8c7-ab57c45dde55.png">

<img width="499" alt="image" src="https://user-images.githubusercontent.com/10085348/183756603-139dc79c-d63e-457e-a265-1ddba1463b69.png">


```
cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yml
```

<img width="1465" alt="image" src="https://user-images.githubusercontent.com/10085348/183757253-c38c33fd-0bcf-4014-b678-9f6fb12a73e6.png">


Make sure that wireshark is deleted on all the servers by running `wireshark --version`


<img width="446" alt="image" src="https://user-images.githubusercontent.com/10085348/183758864-aed67df1-8fab-4ef5-8b27-7041fa283931.png">

<img width="428" alt="image" src="https://user-images.githubusercontent.com/10085348/183759382-145cf05c-5199-4214-888b-35e421457c72.png">

<img width="881" alt="image" src="https://user-images.githubusercontent.com/10085348/183760017-3591ba4e-0968-4e6c-97ad-5a621c0c27cf.png">

<img width="880" alt="image" src="https://user-images.githubusercontent.com/10085348/183760240-96ee5464-89e7-4267-9377-d5019affc122.png">

<img width="880" alt="image" src="https://user-images.githubusercontent.com/10085348/183760470-c547c49b-f29f-4c6a-aaf9-efaaa4ad8343.png">


Now you have learned how to use `import_playbooks module` and you have a ready solution to install/delete packages on multiple servers with just one command.


### Configure UAT Webservers with a role ‘Webserver’

We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.


Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT

<img width="1222" alt="image" src="https://user-images.githubusercontent.com/10085348/183852437-3210a973-514e-4790-ba62-7dcdfadd5462.png">

Use an Ansible utility called `ansible-galaxy` inside `ansible-config-mgt/roles` directory (you need to create roles directory upfront)

<img width="609" alt="image" src="https://user-images.githubusercontent.com/10085348/183854942-d0b8a71d-8519-4641-b9d7-d112b52408b8.png">

The entire folder structure should look like below:

<img width="575" alt="image" src="https://user-images.githubusercontent.com/10085348/183855945-b1f90c18-e16e-4045-a52e-ccd3403df5d6.png">

Remove `tests, files, and vars` since we `ansible-galaxy` utility

<img width="645" alt="image" src="https://user-images.githubusercontent.com/10085348/183856036-90b58983-5c6c-4a91-95ef-3394b60c00bb.png">


setup SSH agent and connect VS Code to your Jenkins-Ansible instance

<img width="689" alt="image" src="https://user-images.githubusercontent.com/10085348/183873618-fd4ac406-ce0f-4143-b382-cd9da7263fbd.png">



Update your inventory `ansible-config-mgt/inventory/uat.yml` file with IP addresses of your 2 UAT Web servers

<img width="494" alt="image" src="https://user-images.githubusercontent.com/10085348/183858168-facdbde6-f430-4348-a74c-e2f346df2bcc.png">

In `/etc/ansible/ansible.cfg` file uncomment `roles_path` string and provide a full path to your roles directory `roles_path    = /home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles.


<img width="469" alt="image" src="https://user-images.githubusercontent.com/10085348/183860493-702cd01b-0ebb-4eb6-9cf7-0b98c90de110.png">


Time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:


Install and configure Apache (httpd service)


Clone Tooling website from GitHub `https://github.com/kebsOps/tooling.git`


Ensure the tooling website code is deployed to `/var/www/html` on each of 2 UAT Web servers.

  
Make sure httpd service is started


Your `main.yml` may consist of following tasks:

<img width="555" alt="image" src="https://user-images.githubusercontent.com/10085348/183871471-e65b6f90-c8ad-413f-ae1c-8dfcc4917f93.png">


### Reference ‘Webserver’ role

Within the static-assignments folder, create a new assignment for uat-webservers `uat-webservers.yml`. This is where you will reference the role.

<img width="705" alt="image" src="https://user-images.githubusercontent.com/10085348/183863403-bb15b766-f120-4c0c-b091-36c42d72bac5.png">

Remember that the entry point to our ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml`

<img width="451" alt="image" src="https://user-images.githubusercontent.com/10085348/183864980-0d8b1d7a-f936-4fc6-a567-0cdbd83fcb90.png">


### Commit & Test

Commit your changes, create a Pull Request and merge them to `master` branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your `Jenkins-Ansible` server into `/home/ubuntu/ansible-config-mgt/` directory.


<img width="1036" alt="image" src="https://user-images.githubusercontent.com/10085348/183866443-862ef16e-66b7-4f3a-aa61-0a908cab602e.png">


<img width="624" alt="image" src="https://user-images.githubusercontent.com/10085348/183866769-9575f079-4941-493c-905e-1c6f28a1ccb6.png">

<img width="746" alt="image" src="https://user-images.githubusercontent.com/10085348/183867715-f70c1650-4f29-4a34-9d29-c698d53698bc.png">

<img width="1308" alt="image" src="https://user-images.githubusercontent.com/10085348/183868476-0157224b-6da1-4faf-a8fc-81d30a0156be.png">

<img width="1095" alt="image" src="https://user-images.githubusercontent.com/10085348/183868975-6db5cdb5-e20c-424d-85e3-3d366f05a060.png">

<img width="1165" alt="image" src="https://user-images.githubusercontent.com/10085348/183869709-2e5d1f02-8218-4842-ad52-a622023b0c3f.png">


<img width="1189" alt="image" src="https://user-images.githubusercontent.com/10085348/183870719-f3e32e50-5ec9-4661-8c8a-e1c501cb23b8.png">


<img width="1057" alt="image" src="https://user-images.githubusercontent.com/10085348/183870390-d4602de6-9fd6-40ef-80fb-36fa21538002.png">


Now run the playbook against your `uat` inventory and see what happens:

```
sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml
```



