## Deploying Applications Into Kubernetes Cluster


Create AWS EKS cluster via the link [eksctl - The official CLI for Amazon EKS](https://github.com/eksctl-io/eksctl)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/ee89e8d1-a9b9-4e52-87f9-5f21481284ea)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/e63efd3b-23e9-4c6c-97ba-6082b2ddfc17)



![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/78b15861-1ccf-45f8-a85e-93ce32e75227)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/43ad4ef3-7829-4e4c-87a4-168e8badd8ca)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/d2054500-ece8-4e77-9b2d-602727ceea01)

### Create ReplicaSet

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/b1e5486a-1c49-4edf-9908-5bf7858fca05)

The manifest file of ReplicaSet consist of the following fields:

- apiVersion: This field specifies the version of kubernetes Api to which the object belongs. ReplicaSet belongs to apps/v1 apiVersion.
- kind: This field specify the type of object for which the manifest belongs to. Here, it is ReplicaSet.
- metadata: This field includes the metadata for the object. It mainly includes two fields: name and labels of the ReplicaSet.
- spec: This field specifies the label selector to be used to select the Pods, number of replicas of the Pod to be run and the container or list of containers which the Pod will run. In the above example, we are running 3 replicas of nginx container.


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/adcc47d8-9c83-483a-ae08-3ffa3276ca92)


### Scaling replicaSets to 5 in the default namespace

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/fa089d1c-d394-4c02-bde3-74c0e40e1ccf)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/66b23eb6-4819-4d79-aeaa-e8c1907df933)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/2f406187-663f-4d93-acb3-071942838ff7)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/9d1c4f43-3e98-4001-9bb4-58bc59e64025)


![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/32a13ad4-a51d-4e1d-a306-2cfc303d49b0)





