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


<img width="1053" alt="image" src="https://user-images.githubusercontent.com/10085348/184549417-f7c7507f-fa9a-4a87-b908-a6fb15b68f35.png">

In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}`
From now on, each time you hit on execute, it will expect an input

<img width="1104" alt="image" src="https://user-images.githubusercontent.com/10085348/184688137-9d0c6c6a-5961-46b2-a6fe-d6c4acbcfefa.png">

<img width="1420" alt="image" src="https://user-images.githubusercontent.com/10085348/184549003-14f83300-133e-4a17-8140-02545456ce9e.png">


### SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION

We already have `tooling` website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from `Artifactory` rather than from git.


