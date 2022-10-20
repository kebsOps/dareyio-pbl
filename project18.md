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

