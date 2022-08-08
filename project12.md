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




