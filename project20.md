## MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER AND DOCKER COMPOSE

Until now, we have been using VMs (AWS EC2) in Amazon Virtual Private Cloud (AWS VPC) to deploy our web solutions, and it works well in many cases.


We have learned how easy to spin up and configure a new EC2 manually or with such tools as Terraform and Ansible to automate provisioning and configuration.


We've also deployed two different websites on the same VM; this approach is scalable, but to some extent; imagine what if we need to deploy many small applications (it can be web front-end, web-backend, processing jobs, monitoring, logging solutions, etc.)
and some of the applications will require various OS and runtimes of different versions and conflicting dependencies – in such case we would need to spin up servers for each group of applications with the exact OS/runtime/dependencies requirements. 
When it scales out to tens/hundreds and even thousands of applications (e.g., when we talk of microservice architecture), this approach becomes very tedious and challenging to maintain.

In this project, we will learn how to solve this problem and practice the technology that revolutionized application distribution and deployment back in 2013! We are talking of Containers and imply Docker. Even though there are other application containerization technologies, Docker is the standard and the default choice for shipping your app in a container!

## Install Docker and prepare for migration to the Cloud

First, we need to install Docker Engine, which is a client-server application that contains:

- A server with a long-running daemon process dockerd.
- APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
- A command-line interface (CLI) client docker.


## Migrate Tooling Web Application from a VM-based solution into a containerized one


### MySQL in container

Pull MySQL Docker Image from Docker Hub Registry

``docker pull mysql/mysql-server:latest``

<img width="758" alt="image" src="https://user-images.githubusercontent.com/10085348/214514092-e3344079-35d9-44ff-a9c6-233fd60ef8a6.png">


### List the images to check that you have downloaded them successfully:

``docker image ls``

![image](https://user-images.githubusercontent.com/10085348/214516135-0719e115-8314-429a-9397-b5589576585f.png)


### Deploy the MySQL Container to your Docker Engine

``docker run --name mysql -e MYSQL_ROOT_PASSWORD=adminPassword -d mysql/mysql-server:latest``

Replace name and password flags to suit your preference. 
Then Run ``docker ps -a to see all your containers (running or stopped)

![image](https://user-images.githubusercontent.com/10085348/214516945-e7e4ffaa-0d92-4929-bb61-d4d530946c5a.png)

<img width="1095" alt="Screenshot 2023-01-25 at 09 36 07" src="https://user-images.githubusercontent.com/10085348/214517275-c96c0110-d39e-4984-91bd-f2a170a238de.png">


The virtual environment status changes from health: starting to healthy, once the setup is complete.


## Connecting to the MySQL Docker container

We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.

**Approach 1**

Connecting directly to the container running the MySQL server:

``docker exec -it mysql bash``

or

``docker exec -it mysql mysql -uroot -p``

Provide the root password when prompted. With that, you’ve connected the MySQL client to the server.

<img width="605" alt="image" src="https://user-images.githubusercontent.com/10085348/214518262-71368a2c-92dc-4dcb-a85c-3acc8d10720d.png">


**Approach 2**

At this stage you are now able to create a docker container but we will need to add a network. So, stop and remove the previous mysql docker container.

```
docker ps -a
docker stop mysql_db
docker rm mysql_db or <container ID> a314ead7bacf
```

<img width="661" alt="image" src="https://user-images.githubusercontent.com/10085348/214519977-2c81104a-1b3d-4637-8d0d-936d4e223f55.png">

First, create a network:

``docker network create --subnet=172.18.0.0/24 tooling_app_network``

<img width="719" alt="image" src="https://user-images.githubusercontent.com/10085348/214521319-8a5a1fa0-30ea-4032-bdf1-df9b6861192f.png">


Export an environment variable containing the root password setup earlier:

``export MYSQL_PW=admin12345``


Then, pull the image and run the container, all in one command like below:

 ``docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest``

<img width="1250" alt="image" src="https://user-images.githubusercontent.com/10085348/214522443-f1d5554f-fed8-49e4-8b10-1d86164e9bd2.png">


Verify the container is running:

<img width="1185" alt="image" src="https://user-images.githubusercontent.com/10085348/214523820-b6e64beb-7c16-48f2-bc4f-63436a503fd8.png">

Create a file and name it create_user.sql and add the below code in the file:

 ``CREATE USER ''@'%' IDENTIFIED BY ''; GRANT ALL PRIVILEGES ON * . * TO ''@'%';``

Run the script:

Ensure you are in the directory  ``create_user.sql`` file is located or declare a path

 ``docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql``

If you see a warning like below, it is acceptable to ignore:

``mysql: [Warning] Using a password on the command line interface can be insecure.``

## Connecting to the MySQL server from a second container running the MySQL client utility

Run the MySQL Client Container:

 ``docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u  -p``
 
 
 ## Prepare database schema
 
Now you need to prepare a database schema so that the Tooling application can connect to it.

- Clone the Tooling-app repository from here

 ``git clone https://github.com/darey-devops/tooling.git``
On your terminal, export the location of the SQL file

 ``export tooling_db_schema=./tooling_db_schema.sql`` 

You can find the ``tooling_db_schema.sql`` in the ``tooling/html/tooling_db_schema.sql`` folder of the cloned repo.

- Verify that the path is exported

 ``echo $tooling_db_schema``
 
- Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.
 
 ``docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema`` 

- Update the ``.env file`` with connection details to the database
 The ``.env file`` is located in the **html** ``tooling/html/.env`` folder but not visible in terminal. you can use vi or nano

```
sudo vi .env

MYSQL_IP=mysqlserverhost
MYSQL_USER=username
MYSQL_PASS=client-secrete-password
MYSQL_DBNAME=toolingdb
```

### Run the Tooling App

- First build the Docker image. change directory into the tooling folder, where the Dockerfile is located:

<img width="906" alt="image" src="https://user-images.githubusercontent.com/10085348/214531236-463c173e-66f9-42a2-b275-4a6020b6887b.png">

Then run:

``docker build -t tooling:0.0.1 . ``


<img width="983" alt="image" src="https://user-images.githubusercontent.com/10085348/214530936-be744f85-4bb9-4a54-9c36-d5ff3b905d2e.png">

_In the above command, we specify a parameter -t, so that the image can be tagged **tooling"0.0.1** - Also, you have to notice the **.** at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile._

Run the container:

``docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1 ``

**Blocker**:

Had to Update ``db_conn.php`` file with the database connection information below:

``
$servername = "mysqlserverhost"; // input servername
$username = ""; // input username
$password = ""; //input password
$dbname = "toolingdb"; // input dbname
``

Then rebuilt the image and ran the container

<img width="1236" alt="Screenshot 2023-01-25 at 11 08 11" src="https://user-images.githubusercontent.com/10085348/214535766-7682757a-b0e9-43cf-ba32-ccc91c17f225.png">

If everything works, you can open the browser and type http://localhost:8085

You will see the login page.

<img width="1153" alt="image" src="https://user-images.githubusercontent.com/10085348/214536132-d373057b-a3ec-447c-a3ae-2fb8b1af897b.png">



## Practice Task № 1 – Implement a POC to migrate the PHP-Todo app into a containerized application.

Download php-todo repository [from here](https://github.com/darey-devops/php-todo)

## Part 1

- Write a Dockerfile for the TODO app

<img width="940" alt="image" src="https://user-images.githubusercontent.com/10085348/215599829-a289fc5e-6c68-4d97-a01c-edf7a5423e83.png">


- Run both database and app on your laptop Docker Engine

<img width="1019" alt="image" src="https://user-images.githubusercontent.com/10085348/215600031-d50dc3d8-e84c-4a4b-a5db-de7bf808eb02.png">


<img width="483" alt="image" src="https://user-images.githubusercontent.com/10085348/215600783-695b39ee-1c72-4995-9f78-f6c475f5c1bf.png">


<img width="799" alt="image" src="https://user-images.githubusercontent.com/10085348/215600243-6b919f89-fa93-46ff-8df4-6c673dc97197.png">


<img width="1037" alt="image" src="https://user-images.githubusercontent.com/10085348/215602401-c942b8cf-5e07-4433-91db-0e26751ea331.png">

<img width="1037" alt="image" src="https://user-images.githubusercontent.com/10085348/215602719-bb07ff13-c092-4125-9b67-55743abf5246.png">


- Access the application from the browser

<img width="1255" alt="image" src="https://user-images.githubusercontent.com/10085348/215603242-49c39bea-691f-4d66-837f-ba82dfc483e3.png">


## Part 2

- Create an account in Docker Hub

<img width="1293" alt="image" src="https://user-images.githubusercontent.com/10085348/215759017-ea994173-79be-4e10-a2cf-c161c2d5e903.png">


- Create a new Docker Hub repository

<img width="1290" alt="image" src="https://user-images.githubusercontent.com/10085348/215764505-b04c6b1e-30e1-4215-a340-1fdd537af210.png">


- Push the docker images from your PC to the repository

<img width="750" alt="image" src="https://user-images.githubusercontent.com/10085348/215766184-4886dce4-faf6-4905-841b-19d78ed0860f.png">

<img width="1264" alt="image" src="https://user-images.githubusercontent.com/10085348/215766403-f95625b5-26d5-4967-b5b1-a6b9fe696624.png">


## Part 3

- Write a Jenkinsfile that will simulate a Docker Build and a Docker Push to the registry

**Setup Jenkins**

 Run the command ``docker pull jenkins/jenkins:lts`` to pull Jenkins image from Docker hub.
 
<img width="585" alt="Screenshot 2023-02-01 at 11 17 25" src="https://user-images.githubusercontent.com/10085348/216016271-3aacb085-8316-481a-89a3-64f6d40c4cb3.png">

Run container using ``docker run -d  -p 8080:8080 -p 50000:50000 -v /your/home:/var/jenkins_home jenkins/jenkins:lts``

<img width="894" alt="image" src="https://user-images.githubusercontent.com/10085348/216019512-bd7a2e75-78d9-4a9a-82a7-35ccab1a16b0.png">


- Connect your repo to Jenkins

- Create a multi-branch pipeline

- Simulate a CI pipeline from a feature and master branch using previously created Jenkinsfile

- Ensure that the tagged images from your Jenkinsfile have a prefix that suggests which branch the image was pushed from. For example, **feature-0.0.1**

- Verify that the images pushed from the CI can be found at the registry.


<img width="1207" alt="image" src="https://user-images.githubusercontent.com/10085348/227723392-b6e3d394-981b-4fe0-aee5-e5c0b6e1582b.png">

<img width="1279" alt="image" src="https://user-images.githubusercontent.com/10085348/227723481-c3579f3b-6313-4f18-ac1f-7cd2e39144c5.png">

<img width="1068" alt="image" src="https://user-images.githubusercontent.com/10085348/227924821-8dbe9d1d-aa82-4086-adfb-3e181a4937e5.png">


## Deployment with Docker Compose

Create a file, name it ``tooling.yaml``

```
version: "3.9"
services:
  tooling_frontend:
    build: .
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
    links:
      - db
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: <The database name required by Tooling app >
      MYSQL_USER: <The user required by Tooling app >
      MYSQL_PASSWORD: <The password required by Tooling app >
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql
volumes:
  tooling_frontend:
  db:

```

Run the command to start the containers

``docker-compose -f tooling.yaml  up -d``

<img width="1156" alt="image" src="https://user-images.githubusercontent.com/10085348/227917848-5ed0f53d-7046-45b6-a205-94f6354d48b8.png">

Verify that the compose is in the running status:

``docker compose ls``

<img width="892" alt="image" src="https://user-images.githubusercontent.com/10085348/227918110-a5ff03df-372c-4f8b-853b-2750546a82bc.png">
 
## Practice Task № 2 – Complete Continous Integration With A Test Stage

## Document your understanding of the various fields in tooling.yaml

- version: "3.9" specifies the version of the Compose file format to use.
- services: defines the services that make up the application.
- tooling_frontend: a service that runs the frontend of the application.
- build: specifies the Dockerfile to use for building the image.
- ports: maps the container port 80 to the host port 5000.
- volumes: mounts the tooling_frontend volume to the /var/www/html directory in the container.
- links: creates a link to the db service.
- db: a service that runs a MySQL database.
- image: specifies the Docker image to use.
- restart: sets the restart policy to "always", so the container restarts automatically if it stops.
- environment: sets environment variables that are required for the database to function properly.
- volumes: mounts the db volume to the `/var/lib/mysql` directory in the container.
- volumes: defines the named volumes used by the services to persist data across container restarts. In this case, two named volumes are defined: tooling_frontend and db.

## Update Jenkinsfile with a test stage

<img width="1278" alt="image" src="https://user-images.githubusercontent.com/10085348/228618140-8d0ae8b8-9a7d-452d-8100-86483bb1d623.png">

[Link to github repo](https://github.com/kebsOps/php-todo)


