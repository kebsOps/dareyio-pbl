### AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 3 – REFACTORING

we will need to configure a backend where the state file can be accessed remotely other DevOps team members. There are plenty of different standard backends supported by Terraform that you can choose from. Since we are already using AWS – we can choose an **S3 bucket** as a backend.

Another useful option that is supported by S3 backend is State Locking – it is used to lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. State Locking feature for S3 backend is optional and requires another AWS service – DynamoDB.

Let us configure it!

### Here is our plan to Re-initialize Terraform to use S3 backend:

- Add S3 and DynamoDB resource blocks before deleting the local state file
- Update terraform block to introduce backend and locking
- Re-initialize terraform
- Delete the local ``tfstate`` file and check the one in S3 bucket
- Add ``outputs``
- ``terraform apply``


Create a file called ``backend.tf`` and input the code below:

<img width="986" alt="image" src="https://user-images.githubusercontent.com/10085348/196889429-34990847-4096-4254-b77c-9ce46164a750.png">



<img width="518" alt="image" src="https://user-images.githubusercontent.com/10085348/196889561-c2ad0c89-77ea-480e-b781-827a0345af00.png">



<img width="947" alt="image" src="https://user-images.githubusercontent.com/10085348/194719085-2a4790dc-75a1-4e79-833c-f3244bfb6d17.png">


<img width="1018" alt="image" src="https://user-images.githubusercontent.com/10085348/196970351-474d3e39-5023-4d12-8245-30cd5f33a505.png">

Configure S3 Backend

<img width="611" alt="image" src="https://user-images.githubusercontent.com/10085348/196890156-805d134b-4341-42fa-a661-6ea499ff3117.png">


<img width="936" alt="image" src="https://user-images.githubusercontent.com/10085348/196987210-18686897-3de0-4e47-a702-1fe49206af2a.png">


<img width="1493" alt="image" src="https://user-images.githubusercontent.com/10085348/196998916-cc211063-a1c6-4fb2-9ca3-1b1cd56a4a7b.png">


<img width="1507" alt="image" src="https://user-images.githubusercontent.com/10085348/196970902-10cec319-eb86-4303-941f-a74adbfd9e59.png">


<img width="1401" alt="image" src="https://user-images.githubusercontent.com/10085348/197000282-ca48f6ce-e77e-4a1b-b4bc-790882c7c326.png">


## Refactoring Code using Modules

In continuation to [Project 17](https://github.com/kebsOps/dareyio-pbl/blob/main/project17.md), the entire code will refactored order to improve its readability and reusability using a Terraform tool called Module.

The following outlines detailed step taken to achieve this:

-  Create a folder called **modules** then the following subfolders:

    - **ALB**: For Aplication Load balancer and similar resources
    - **EFS**: For Elastic file system resources
    - **RDS**: For Databases resources
    - **Autoscaling**: For Autosacling and launch template resources
    - **compute**: For EC2 and rlated resources
    - **VPC**: For VPC and netowrking resources such as subnets, roles, e.t.c.
    - **security**: for creating security group resources
  
-  Each module shall contain following files:

   - **main.tf** (or ``%resource_name%.tf``) file(s) with resources blocks
   - **outputs.tf** (optional, if you need to refer outputs from any of these resources in your root module)
   - **variables.tf** (as we learned before - it is a good practice not to hard code the values and use variables)

  

## PBL Folder Structure

<img width="240" alt="image" src="https://user-images.githubusercontent.com/10085348/199025316-d25e5446-5345-401f-872c-ff13e6b57dff.png">


[Link to complete code repository on github](https://github.com/kebsOps/Terraform/tree/project-18)

## Refactoring Code for VPC folder:

``output.tf``

<img width="814" alt="image" src="https://user-images.githubusercontent.com/10085348/199030317-46605efe-ee86-4c4d-bb40-caa0aed408fb.png">



``variables.tf``

<img width="965" alt="image" src="https://user-images.githubusercontent.com/10085348/199029188-16d05541-49aa-4dff-bbc7-7b83665a2aa3.png">

<img width="998" alt="image" src="https://user-images.githubusercontent.com/10085348/199029287-ca0f768a-30d6-4748-b343-734b9bf82888.png">


## Refactoring Code for security folder:

``main.tf``

<img width="717" alt="image" src="https://user-images.githubusercontent.com/10085348/199029512-3c65751e-c43a-4eb6-b311-7b653a54bacf.png">


## Refactoring Code for ALB folder:

``output.tf``

<img width="732" alt="image" src="https://user-images.githubusercontent.com/10085348/199029892-9841bf03-c5a5-4da8-a7f2-7f37f71ddb75.png">


## Executing Terraform Plan


- To ensure the validation of the whole setup, run the command `terraform validate`

<img width="559" alt="image" src="https://user-images.githubusercontent.com/10085348/199024189-6db21e12-5b47-430c-94bf-3a1029d8aa5a.png">


<img width="1197" alt="image" src="https://user-images.githubusercontent.com/10085348/199021498-19842899-7d7f-49bc-86f6-4dd6c1a11ead.png">


<img width="1019" alt="image" src="https://user-images.githubusercontent.com/10085348/199021599-578fb398-9a28-49d9-8786-b7e6804d55d3.png">


<img width="1018" alt="image" src="https://user-images.githubusercontent.com/10085348/196970351-474d3e39-5023-4d12-8245-30cd5f33a505.png">


<img width="1101" alt="image" src="https://user-images.githubusercontent.com/10085348/199021749-7962de08-ef1e-4233-a188-72ddf4d87f04.png">

<img width="1207" alt="image" src="https://user-images.githubusercontent.com/10085348/199021893-abb16576-c30c-44cf-a359-277529643c48.png">


<img width="1018" alt="image" src="https://user-images.githubusercontent.com/10085348/196970351-474d3e39-5023-4d12-8245-30cd5f33a505.png">
