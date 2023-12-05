## Deploying and Packaging applications into Kubernetes with Helm

In [project 24](https://github.com/kebsOps/dareyio-pbl/blob/main/Project24.md), you started experiencing helm as a tool used to deploy an application into Kubernetes. You probably also tried installing more tools apart from Jenkins.

In this project, you will experience deploying more DevOps tools, get familiar with some of the real world issues faced during such deployments and how to fix them. You will learn how to tweak helm values files to automate the configuration of the applications you deploy. Finally, once you have most of the DevOps tools deployed, you will experience using them and relate with the DevOps cycle and how they fit into the entire  ecosystem.

Our focus will be on the. 

- Artifactory
- Ingress Controllers
- Cert-Manager

Then

- Prometheus
- Grafana
- Elasticsearch ELK using [ECK](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-install-helm.html)

For the tools that require paid license, don't worry, you will also learn how to get the license for free and have true experience exactly how they are used in the real world.

Lets start first with Artifactory. What is it exactly?

Artifactory is part of a suit of products from a company called [Jfrog](https://jfrog.com/). Jfrog started out as an artifact repository where software binaries in different formats are stored. Today, Jfrog has transitioned from an artifact repository to a DevOps Platform that includes CI and CD capabilities. This has been achieved by offering more products in which **Jfrog Artifactory** is part of. Other offerings include 
      
  - __JFrog Pipelines__ -  a CI-CD product that works well with its Artifactory repository. Think of this product as an alternative to Jenkins.
  - __JFrog Xray__ - a security product that can be built-into various steps within a JFrog pipeline. Its job is to scan for security vulnerabilities in the stored artifacts. It is able to scan all dependent code.

In this project, the requirement is to use Jfrog Artifactory as a private registry for the organisation's Docker images and Helm charts. This requirement will satisfy part of the company's corporate security policies to never download artifacts directly from the public into production systems. We will eventually have a CI pipeline that initially pulls public docker images and helm charts from the internet, store in artifactory and scan the artifacts for security vulnerabilities before deploying into the corporate infrastructure. Any found vulnerabilities will immediately trigger an action to quarantine such artifacts.

Lets get into action and see how all of these work.

## Deploy Jfrog Artifactory into Kubernetes

The best approach to easily get Artifactory into kubernetes is to use helm.

Search for an official helm chart for Artifactory on [Artifact Hub](https://artifacthub.io/)
   
<img width="1479" alt="Screenshot 2023-12-05 at 15 34 01" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/d6b2eb3c-e542-422b-9079-834bc4f697c1">

Click on install to see installation instructions.

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/684fb618-1cd8-4586-95af-d988e81fe5bb)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/15ca6c29-2a2f-423b-b288-f92869e30551)

Add the repo

`helm repo add jfrog https://charts.jfrog.io`

Update the helm repo index on my local machine/laptop

`helm repo update`

Install artifactory in the namespace __tools__

`helm upgrade --install artifactory jfrog/artifactory --version 107.71.5 -n tools`
