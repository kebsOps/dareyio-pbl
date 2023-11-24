## Building Elastic Kubernetes Service (EKS) With Terraform - Deploying and Packaging applications with Helm

This project seeks to solidify your skills by focusing more on best practice set up:

- You will use Terraform to create a Kubernetes EKS cluster and dynamically add scalable worker nodes
- You will deploy multiple applications using HELM
- You will experience more kubernetes objects and how to use them with Helm.
- You will improve upon your CI/CD skills with Jenkins

**Note:** Use Terraform version v1.0.2

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/240b5211-1550-4026-8c83-39dd7edb3bfe)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bcc0a529-d156-40a7-9ee0-b30b524c2aa1)

 
Open up a new directory on your laptop, and name it ``EKS`` Use AWS CLI to create an S3 bucket

Create a file – ``backend.tf`` Task for you, ensure the backend is configured for remote state in S3
