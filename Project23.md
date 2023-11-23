## Persisting Data in Kubernetes

In Kubernetes, pods are transient and typically have a short lifespan. If a pod is terminated, any data not included in its container image is lost upon restart. This is because Kubernetes excels in managing stateless applications, where data persistence is not inherently managed. To maintain data across pod restarts and ensure its persistence, Kubernetes employs the PersistentVolume resource. This resource is designed to handle data storage and ensure that data remains intact even when pods are cycled.

This project introduces the concept of data persistence in Kubernetes, focusing on Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and Storage Classes.

- **A PersistentVolume (PV)** represents a segment of storage within the cluster, provisioned either by an administrator or dynamically through Storage Classes. Similar to how a node is a resource in the cluster, a PV is also a cluster resource. It differs from regular volume plugins in its lifecycle, which is independent of any specific pod utilizing the PV. This API object details the storage's implementation, whether it's NFS, iSCSI, a cloud-provider-specific storage system, or others.

- **PersistentVolumeClaims (PVCs)** serve as a user's request for storage, akin to how a pod operates. Just as pods utilize node resources, PVCs utilize PV resources. While pods can request specific levels of resources like CPU and memory, PVCs can request specific sizes and access modes. These access modes include ReadWriteOnce, ReadOnlyMany, or ReadWriteMany, reflecting the variety of ways a PVC can be mounted.

To address the need for PersistentVolumes with diverse properties such as performance, StorageClass resources are used. These cater to a range of PersistentVolumes that vary not just in size and access modes, but in more complex aspects, without burdening users with the intricacies of their implementation. This is particularly useful for cluster administrators who need to offer a spectrum of PersistentVolumes tailored for different requirements.



