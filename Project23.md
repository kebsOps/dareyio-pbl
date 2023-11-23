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


### Check logs of pod

![image](https://github.com/kebsOps/dareyio-pbl/assets/10085348/4a425479-1f54-457e-83bc-7ef8458d182f)


### Exec into the kubernetes pod and confirm that nginx pod default.conf file exist

<img width="1015" alt="Screenshot 2023-11-23 at 13 30 24" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/bc1228c0-6fc0-4071-b448-363e3def3f33">

### 

<img width="1238" alt="Screenshot 2023-11-23 at 13 44 12" src="https://github.com/kebsOps/dareyio-pbl/assets/10085348/678b09fa-32ac-428e-9551-92d3ea3ad8a7">



