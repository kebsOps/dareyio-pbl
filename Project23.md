## Persisting Data in Kubernetes

In Kubernetes, pods are transient and typically have a short lifespan. If a pod is terminated, any data not included in its container image is lost upon restart. This is because Kubernetes excels in managing stateless applications, where data persistence is not inherently managed. To maintain data across pod restarts and ensure its persistence, Kubernetes employs the PersistentVolume resource. This resource is designed to handle data storage and ensure that data remains intact even when pods are cycled.

This project introduces the concept of data persistence in Kubernetes, focusing on Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and Storage Classes.

- **A PersistentVolume (PV)** represents a segment of storage within the cluster, provisioned either by an administrator or dynamically through Storage Classes. Similar to how a node is a resource in the cluster, a PV is also a cluster resource. It differs from regular volume plugins in its lifecycle, which is independent of any specific pod utilizing the PV. This API object details the storage's implementation, whether it's NFS, iSCSI, a cloud-provider-specific storage system, or others.

- **PersistentVolumeClaims (PVCs)** serve as a user's request for storage, akin to how a pod operates. Just as pods utilize node resources, PVCs utilize PV resources. While pods can request specific levels of resources like CPU and memory, PVCs can request specific sizes and access modes. These access modes include ReadWriteOnce, ReadOnlyMany, or ReadWriteMany, reflecting the variety of ways a PVC can be mounted.

To address the need for PersistentVolumes with diverse properties such as performance, StorageClass resources are used. These cater to a range of PersistentVolumes that vary not just in size and access modes, but in more complex aspects, without burdening users with the intricacies of their implementation. This is particularly useful for cluster administrators who need to offer a spectrum of PersistentVolumes tailored for different requirements.


### Prerequisites

- Setup an EKS Cluster on AWS
- Configure both cluster role and node group roles

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/9c8b2362-9116-4f6f-bee9-e3ecdd64ef2e)

### View Deployment 

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/5b2b3294-102b-408b-b419-8e72aa44180f)


#### Details of one of the running pods

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/3cf7b8bd-2ced-47f8-94cc-4feff8289a3a)


### Check logs of one the pods

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/4a425479-1f54-457e-83bc-7ef8458d182f)


### Exec into the kubernetes pod and confirm that nginx pod default.conf file exist

<img width="1015" alt="Screenshot 2023-11-23 at 13 30 24" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/bc1228c0-6fc0-4071-b448-363e3def3f33">

### View details of cluster node, availability zone in which the first node got deployed into

<img width="1238" alt="Screenshot 2023-11-23 at 13 44 12" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/678b09fa-32ac-428e-9551-92d3ea3ad8a7">


### Creating a storage volume within AWS's Elastic Block Storage, ensuring it's in the same Availability Zone as the node operating the Nginx pod. This volume will then be mounted into the Nginx pod

<img width="932" alt="Screenshot 2023-11-23 at 14 02 20" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/5c120c4b-340b-413d-acf7-3b86708c1565">

<img width="1294" alt="Screenshot 2023-11-23 at 14 09 26" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/e29aa98c-b501-4373-a8d1-260b12d809c6">


### Update the deployment configuration with the volume spec and volume mount:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      volumes:
      - name: nginx-volume
        # This AWS EBS volume must already exist.
        awsElasticBlockStore:
          volumeID: "vol-0e29899182fd61b9a"
          fsType: ext4
```

<img width="1240" alt="Screenshot 2023-11-23 at 14 51 11" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/5fac1c55-7a18-4178-abc6-41faaac13fd1">


### Dynamically Handling Volumes Using PVs and PVCs:

PVs serve as cluster resources, while PVCs request these resources and function as claims to them. In EKS, a default storageClass is set up during installation, enabling dynamic PV creation for Pod usage.

If there is no storage class in your cluster, below manifest is an example of how one would be created:

```
  kind: StorageClass
  apiVersion: storage.k8s.io/v1
  metadata:
    name: gp2
    annotations:
      storageclass.kubernetes.io/is-default-class: "true"
  provisioner: kubernetes.io/aws-ebs
  parameters:
    type: gp2
    fsType: ext4 

```

To check if a storageClass exists in the cluster: ``Use $ kubectl get storageclass``

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/0059fffd-014b-47b6-8f92-66573290b03f)


Creating a manifest file for a PVC, and based on the gp2 storageClass a PV will be dynamically created:

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/bd50b7b8-739f-4ca0-84a8-2fdead56489d)

The PVC created is in pending state because PV is not created yet. Editing the nginx-pod.yaml file to create the PV:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-volume-claim
          mountPath: /tmp/kebe
      volumes:
      - name: nginx-volume-claim
        persistentVolumeClaim:
          claimName: nginx-volume-claim
```

### Persisting configuration data with configMaps


Accessing the default homepage of the deployed Nginx server

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/95f7b681-4128-49af-979f-074ed21c1b8a)


- Now the index.html file is no longer ephemeral because it is using a configMap that has been mounted onto the filesystem. This is now evident when you exec into the pod and list the /usr/share/nginx/html directory:
  
![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/d1ad6030-ffb4-4ed8-9caa-7ff0b6f5686e)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/118f25ce-2c15-496a-abd2-0a7553f12836)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/f9476284-4216-467d-a82c-44afc3b2b453)

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/43b2b7d3-fe85-43e7-a87b-99ad589adaf9)

[Link to Project Code](https://github.com/kebsOps/Deploying-Apps-into-kubernetes-cluster-and-Persisting-Data-in-Kubernetes)
