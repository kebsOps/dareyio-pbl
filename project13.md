## PROJECT 13

## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

### In this project we will introduce Ansible dynamic assignments by using include module

This project builds on the previous [Project 12](https://github.com/kebsOps/dareyio-pbl/blob/main/project12.md)

Imports is static and Include is dynamic. 

when include module is used, all statements are processed only during execution of the playbook

When the import module is used, all statements are pre-processed at the time playbooks are parsed. Changes made during playbook execution are ignored.

However, you can use dynamic assignments for environment specific variables as we will be introducing in this project


In https://github.com/kebsOps/ansible-config-mgt GitHub repository start a new branch and call it `dynamic-assignments`.

Create a new folder, name it `dynamic-assignments`. Then inside this folder, create a new file and name it `env-vars.yml`.
We will instruct `site.yml` to include this playbook later. For now, let us keep building up the structure.

![image](https://user-images.githubusercontent.com/10085348/184195440-ae6be08a-1e1a-4597-b302-ec0ff8db80a8.png)


Your GitHub shall have following structure by now:

<img width="784" alt="image" src="https://user-images.githubusercontent.com/10085348/184194688-f9b10b37-8d7c-43b3-b059-9294dd07de24.png">


Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes,
such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. 
Therefore, create a new folder `env-vars`, then for each environment, create new YAML files which we will use to set variables.

<img width="560" alt="image" src="https://user-images.githubusercontent.com/10085348/184196175-bf6df6a1-09bd-4d20-a62e-243c2e41b0cb.png">

Your layout should now look like this:

<img width="761" alt="image" src="https://user-images.githubusercontent.com/10085348/184195067-b76bcf32-d80d-44b3-9602-312bc5050d9f.png">

Now update the `env-vars.yml` file.

<img width="689" alt="image" src="https://user-images.githubusercontent.com/10085348/184196010-0c555497-afb9-4058-af5a-6893d9ec588a.png">

We used `include_vars` syntax instead of include, this is because Ansible developers decided to separate different features of the module.

We made use of a `special variables` `{ playbook_dir }` and `{ inventory_file }`. `{ playbook_dir }` will help Ansible to determine 
the location of the running playbook, and from there navigate to other path on the filesystem. 
`{ inventory_file }` on the other hand will dynamically resolve to the name of the inventory file being used, then append `.yml` 
so that it picks up the required file within the `env-vars` folder.

We are including the variables using a loop. `with_first_found` implies that, looping through the list of files, the first one found is used.
This is good so that we can always set default values in case an environment specific env file does not exist.


### Update site.yml with dynamic assignments

Update `site.yml` file to make use of the dynamic assignment. (_At this point, we cannot test it yet. 
We are just setting the stage for what is yet to come. So hang on to your hats_)

`site.yml` should now look like this.

<img width="481" alt="image" src="https://user-images.githubusercontent.com/10085348/184198925-036dbaa1-0948-424b-a06d-cbe80cf84c10.png">

### Community Roles

Now it is time to create a role for MySQL database – it should install the MySQL package, 
create a database and configure users. But why should we re-invent the wheel? 
There are tons of roles that have already been developed by other open source engineers out there. 
These roles are actually production ready, and dynamic to accomodate most of Linux flavours.
With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

### Download Mysql Ansible Role

You can browse available community roles [here](https://galaxy.ansible.com/home)

We will be using a [MySQL role developed by geerlingguy](https://galaxy.ansible.com/geerlingguy/mysql)

<img width="773" alt="image" src="https://user-images.githubusercontent.com/10085348/184208115-4727bafd-cc98-433c-a158-c260eac3ae99.png">

Hint: To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ` ansible-config-mgt ` directory.
Of course you must have `git` installed and configured on `Jenkins-Ansible` server and, for more convenient work with codes, you can configure Visual Studio Code
to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on `Jenkins-Ansible` server, so you can disable it – we will be using Jenkins later for a better purpose.

On Jenkins-Ansible server make sure that git is installed with `git --version`, then go to `ansible-config-mgt` directory and run

<img width="580" alt="image" src="https://user-images.githubusercontent.com/10085348/184209237-914ba2f3-754f-4a95-88ad-37cbf34c9381.png">

Rename the `geerlingguy.mysql` folder to `mysql`

<img width="637" alt="image" src="https://user-images.githubusercontent.com/10085348/184209899-f9216748-84fd-4523-82ce-876fde17c01d.png">

Read `README.md` file, and edit roles configuration to use correct credentials for MySQL required for the `tooling` website.

<img width="521" alt="image" src="https://user-images.githubusercontent.com/10085348/184219812-06c7b2bf-9c80-468c-9c94-7bdda685b21d.png">


<img width="598" alt="image" src="https://user-images.githubusercontent.com/10085348/184219715-13f0e916-2009-4654-a651-cf4c72bda8f8.png">

Now it is time to upload the changes into your GitHub:

<img width="902" alt="image" src="https://user-images.githubusercontent.com/10085348/184220612-bcf3811e-c7d3-4997-a259-1d58a64776e4.png">


<img width="809" alt="image" src="https://user-images.githubusercontent.com/10085348/184220684-20912ed0-ae53-46c6-817f-426779135548.png">

Now, if you are satisfied with your codes, you can create a Pull Request and merge it to `main` branch on GitHub.

<img width="1021" alt="image" src="https://user-images.githubusercontent.com/10085348/184223670-6f7ceeba-4b2d-4b10-8bbc-e3b248b52cf9.png">

<img width="1052" alt="image" src="https://user-images.githubusercontent.com/10085348/184223859-911094ba-7537-4940-8461-6c7073d3cec8.png">


### Load Balancer roles

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

Nginx

Apache


Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.

Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)

```
mkdir roles
cd roles
ansible-galaxy init nginx
ansible-galaxy init apache
```


Declare a variable in `defaults/main.yml` file inside the `nginx` and `apache` roles. Name each variables `enable_nginx_lb` and `enable_apache_lb` respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

<img width="621" alt="image" src="https://user-images.githubusercontent.com/10085348/184406174-d6d48bac-5397-4880-946d-8f9c61754e0e.png">

<img width="611" alt="image" src="https://user-images.githubusercontent.com/10085348/184406319-5231f2ee-fcc3-445d-ac2f-e38a2daa4866.png">

Update both `static-assignments directory file (loadbalancers.yml)` and `site.yml` files respectively

With `when condition` It was used to decide the nature of load balancer installed in an environment depending on which variables is set to true
see the code snippet below:

<img width="651" alt="image" src="https://user-images.githubusercontent.com/10085348/184406573-756132b4-3478-40cd-8355-99757098ddc1.png">

<img width="569" alt="image" src="https://user-images.githubusercontent.com/10085348/184406873-a1286573-0ce0-46a0-9566-e81697de52d2.png">


Now you can make use of `env-vars\uat.yml` file to define which loadbalancer to use in UAT environment by setting respective environmental variable to `true`.

You will activate load balancer, and enable nginx by setting these in the respective environment’s `env-vars` file.


<img width="525" alt="image" src="https://user-images.githubusercontent.com/10085348/184407754-08e484e7-f969-485d-8620-4b3afbe736c9.png">

The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

<img width="919" alt="image" src="https://user-images.githubusercontent.com/10085348/184409833-c664fcc4-bb70-47b5-b6e8-62de91591c59.png">

<img width="989" alt="image" src="https://user-images.githubusercontent.com/10085348/184410179-e7019d71-89dc-4a2e-aaa3-922b1bbe06a3.png">




<img width="717" alt="image" src="https://user-images.githubusercontent.com/10085348/184409545-39c83815-615b-4ad8-b24a-e09d5a41391c.png">

[Project 13 GitHub Repo](https://github.com/kebsOps/ansible-config-mgt/tree/ansible-config-mgt-git-project13)

