## PROJECT 14

## EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP


### What is Continuous Integration?
In software engineering, Continuous Integration (CI) is a practice of merging all developers’ working copies to a shared mainline (e.g., Git Repository or some other version control system) several times per day. Frequent merges reduce chances of any conflicts in code and allow to run tests more often to avoid massive rework if something goes wrong. This principle can be formulated as Commit early, push often.

### Continuous Integration in The Real World
<img width="899" alt="image" src="https://user-images.githubusercontent.com/10085348/184483410-ed0eeca3-e381-480f-94ea-9c857ae73cb6.png">

### Common Best Practices of CI/CD

Principles that define a reliable and robust CI/CD pipeline:

```
- Maintain a code repository
- Automate build process
- Make builds self-tested
- Everyone commits to the baseline every day
- Every commit to baseline should be built
- Every bug-fix commit should come with a test case
- Keep the build fast
- Test in a clone of production environment
- Make it easy to get the latest deliverables
- Everyone can see the results of the latest build
- Automate deployment (if you are confident enough in your CI/CD pipeline and willing to go for a fully automated Continuous Deployment)
```

### TASKS

<img width="914" alt="image" src="https://user-images.githubusercontent.com/10085348/184486324-2092177f-0222-4019-8adf-f95b7395e158.png">


<img width="623" alt="image" src="https://user-images.githubusercontent.com/10085348/184484259-a6a0bef7-e5fa-4b25-a0bb-f16ad5991f93.png">

<img width="731" alt="image" src="https://user-images.githubusercontent.com/10085348/184483045-8e123742-8d4e-4add-abfd-e2c099cf66c7.png">

<img width="721" alt="image" src="https://user-images.githubusercontent.com/10085348/184484681-a62f4cb0-73c8-4f42-bce6-00c576c63e4f.png">

<img width="891" alt="image" src="https://user-images.githubusercontent.com/10085348/184484981-61f9da9e-7eed-4421-b815-0ac8c316ec15.png">

<img width="1249" alt="image" src="https://user-images.githubusercontent.com/10085348/184485355-5e9e80f1-d6f5-4f1f-99ea-9f247f694d7b.png">


<img width="764" alt="image" src="https://user-images.githubusercontent.com/10085348/184487021-a2346eef-8981-4915-bb76-d1e443cd20ff.png">

<img width="1045" alt="image" src="https://user-images.githubusercontent.com/10085348/184487262-ea91a2c2-6616-46f2-989c-5bc83b193785.png">

<img width="1435" alt="image" src="https://user-images.githubusercontent.com/10085348/184485580-2956f77c-f8e9-4e63-9eca-0ed8a9ac6346.png">


### Running Ansible Playbook from Jenkins

Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:


1. Installing Ansible plugin in Jenkins UI
2. Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

Note: Ensure that Ansible runs against the Dev environment successfully.

Parameterizing Jenkinsfile For Ansible Deployment
To deploy to other environments, we will need to use parameters.

Update `sit` inventory with new servers

<img width="424" alt="image" src="https://user-images.githubusercontent.com/10085348/184674081-64d21afc-d16b-440f-8294-2fa98bdd8f50.png">

Update `Jenkinsfile` to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.


![image](https://user-images.githubusercontent.com/10085348/184938419-8a3298e2-7088-451c-aaf8-304e3e5814ff.png)

In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}`
From now on, each time you hit on execute, it will expect an input

<img width="1104" alt="image" src="https://user-images.githubusercontent.com/10085348/184688137-9d0c6c6a-5961-46b2-a6fe-d6c4acbcfefa.png">

<img width="1420" alt="image" src="https://user-images.githubusercontent.com/10085348/184549003-14f83300-133e-4a17-8140-02545456ce9e.png">


### SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION

We already have `tooling` website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from `Artifactory` rather than from git.

Install Configure Artifactory (_Spin up a new Ubuntu 20.04 EC2 Instance_)

```
sudo apt update

sudo apt install gnupg2 -y

wget -qO - https://api.bintray.com/orgs/jfrog/keys/gpg/public.key | sudo apt-key add -

sudo echo "deb https://jfrog.bintray.com/artifactory-debs bionic main" | sudo tee /etc/apt/sources.list.d/jfrog.list

sudo apt update

sudo apt install jfrog-artifactory-oss -y
```

<img width="1149" alt="image" src="https://user-images.githubusercontent.com/10085348/184970601-99f29054-2291-4f2f-b5e5-8ec2149a4aa0.png">


### Start the Artifactory service and enable it to start at system reboot with the following command

<img width="784" alt="image" src="https://user-images.githubusercontent.com/10085348/185395922-d4bd1a70-b8bc-485a-83b7-992e88a34354.png">

Add port 8082 to the ec2 instance security group

<img width="1172" alt="image" src="https://user-images.githubusercontent.com/10085348/185397571-9f6130f0-5149-463c-8e26-0f96e34bff77.png">


<img width="942" alt="image" src="https://user-images.githubusercontent.com/10085348/185399046-144834f1-328d-4522-b63b-92135fa386c5.png">


<img width="1400" alt="image" src="https://user-images.githubusercontent.com/10085348/185399606-22e3c7f5-e6db-47f1-b8cc-df47a4632e43.png">



### Prepare Jenkins
 
Fork the php-todo repository ``https://github.com/darey-devops/php-todo.git``

Install the following packages:

```
sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}
```
 
### Integrate Artifactory repository with Jenkins

Create a dummy `Jenkinsfile` in the repository

Using Blue Ocean, create a multibranch Jenkins pipeline

On the database server, create database and user


```
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```

Remember to update ``.env.sample`` file with  ip address, in our case ``DB_HOST=172.31.46.231``  

Update Jenkinsfile with proper configuration


<img width="963" alt="image" src="https://user-images.githubusercontent.com/10085348/185738262-7c34c762-03b2-48b4-9e98-6a64084df81b.png">


<img width="1430" alt="image" src="https://user-images.githubusercontent.com/10085348/185738316-06fa9610-6cc2-4fd0-8287-eb3dab4bb4cf.png">

Plot the data using plot Jenkins plugin and bundle the application code for into an artifact (archived package) upload to Artifactory

```

    stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit --log-junit reports/unitreport.xml'
      }
    }
    stage('Code Analysis') {
      steps {
          sh 'phploc app/ --log-csv build/logs/phploc.csv'

    }
  }


    stage('Publish & Plot Code Coverage Report') {
     steps {
          sh 'phploc app/ --log-csv build/logs/phploc.csv'

          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'


      }
    }
  stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
   }
  }
}
```
Install xdebug and configure the ``.ini`` file, you can get the path to the file by running

```
php --ini | grep xdebug

And paste in:

xdebug.mode=coverage
```
click on the Plots button on the left pane. If configured properly, you should get output below:

<img width="1066" alt="image" src="https://user-images.githubusercontent.com/10085348/187223881-95d110c5-e571-4bd1-a2b8-74b1b39333c3.png">


<img width="1446" alt="image" src="https://user-images.githubusercontent.com/10085348/185739070-81e30f1a-3c7d-462d-97be-fce3e50548d9.png">

Publish the resulted artifact into Artifactory

<img width="539" alt="image" src="https://user-images.githubusercontent.com/10085348/185746752-2355c9fe-5397-4661-9c20-c3a492c38c21.png">

<img width="1433" alt="image" src="https://user-images.githubusercontent.com/10085348/185746954-d80a9f53-132f-4906-89dd-a2e32d8fd18c.png">


Deploy the application to the `dev` environment by launching Ansible pipeline

<img width="1069" alt="image" src="https://user-images.githubusercontent.com/10085348/185749374-c29304c9-99f0-4899-ae67-17b7e35f16eb.png">

<img width="1292" alt="image" src="https://user-images.githubusercontent.com/10085348/185749385-7ae6a71c-d5a9-4bbc-a81f-672775740637.png">


### SONARQUBE INSTALLATION

we need to implement  **Quality Gate** to ensure that ONLY code with the required code coverage, and other quality standards make it through to the environments.

To achieve this, we need to configure **SonarQube** – An open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities.

**Software Quality** – The degree to which a software component, system or process meets specified requirements based on user needs and expectations.

**Software Quality Gates** – Quality gates are basically acceptance criteria which are usually presented as a set of predefined quality criteria that a software development project must meet in order to proceed from one stage of its lifecycle to the next one.

_SonarQube is a tool that can be used to create quality gates for software projects, and the ultimate goal is to be able to ship only quality software code._


<img width="772" alt="image" src="https://user-images.githubusercontent.com/10085348/185752333-73169098-6ace-4b48-b760-1ad43e87e562.png">


Add **Quality Gate** stage in `Jenkinsfile`

<img width="1027" alt="image" src="https://user-images.githubusercontent.com/10085348/185753178-f541ad90-09a0-4fe1-a210-aab76164610f.png">

Configure sonar-scanner.properties – From the step above, Jenkins will install the scanner tool on the Linux server. You will need to go into the tools directory on the server to configure the properties file in which SonarQube will require to function during pipeline execution.

``cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf``

<img width="977" alt="image" src="https://user-images.githubusercontent.com/10085348/185847194-dd7f854f-e017-49eb-b335-91d162de6837.png">


For this run successfully install [Nodejs 14](https://computingforgeeks.com/install-node-js-14-on-ubuntu-debian-linux/)

<img width="363" alt="image" src="https://user-images.githubusercontent.com/10085348/185755282-792dffc7-6c5e-48ca-b1f7-c0d85a56f91d.png">


<img width="1359" alt="image" src="https://user-images.githubusercontent.com/10085348/185755376-008c3ef8-a72a-4a28-b148-72214b9be9b6.png">

Sonarqube Project page

<img width="1236" alt="image" src="https://user-images.githubusercontent.com/10085348/185755636-9dfe4589-b2e4-4534-aaaa-55866d286030.png">



<img width="774" alt="image" src="https://user-images.githubusercontent.com/10085348/187205001-ff9d45c0-92c0-4343-bbc7-7d72d2c386f7.png">


For specific branches (Quality Gate does not execute I am running from 'feature/ci-pipeline' branch):

<img width="1352" alt="image" src="https://user-images.githubusercontent.com/10085348/185756061-41376138-5830-410e-b818-2e4123ecff48.png">


As a DevOps engineer working on the pipeline, we must ensure that the quality gate step causes the pipeline to fail if the conditions for quality are not met.

<img width="1463" alt="image" src="https://user-images.githubusercontent.com/10085348/187207748-c67fcfde-36b0-4200-aeb1-9bbcd56fe6e5.png">

Quality Gate failed on SonarQube UI

<img width="737" alt="image" src="https://user-images.githubusercontent.com/10085348/187207956-21154ee0-0c63-405c-b4ce-e81c11a421ce.png">


### Configure Jenkins slave servers

Introduce Jenkins agents/slaves – Add 2 more servers to be used as Jenkins slave.

Install the following packages on ``Jenkins-slave-1`` and ``Jenkins-slave-2 Servers``:

```
sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}
```

<img width="1270" alt="image" src="https://user-images.githubusercontent.com/10085348/187252102-1b29e079-e432-4528-a23b-7b9543a3c73c.png">

Blockers: You have to install PHP Composer manually by:

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
mv composer.phar /usr/local/bin/composer
```
Configure Jenkins to run its pipeline jobs randomly on any available slave nodes
 
<img width="1203" alt="image" src="https://user-images.githubusercontent.com/10085348/187252216-f2521f6a-cb3d-4da9-b7b8-26f4159c03ea.png">

Deploy to all Environments

<img width="1213" alt="image" src="https://user-images.githubusercontent.com/10085348/187253982-4f50d416-bcd6-44b5-be5b-7ca0392b17d7.png">


<img width="1482" alt="image" src="https://user-images.githubusercontent.com/10085348/187253285-765e3613-e687-42b3-9da3-dba2c443a47a.png">

<img width="1471" alt="image" src="https://user-images.githubusercontent.com/10085348/187253732-cf4e10fd-9904-4728-acb9-57dcf436e5f5.png">


[Link to pipeline video](https://drive.google.com/file/d/10MFvjn_uOZ349twTthfKsAtsXedNK9t1/view?usp=sharing)

[Link to PHP Todo Repo on github](https://github.com/kebsOps/php-todo)

