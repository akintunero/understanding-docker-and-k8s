
# Day 9: Storage and Volumes

Welcome to Day 9 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Kubernetes Storage and Volumes**, which allow Pods to persist data beyond their lifecycle and share data between containers.

---

## **Why Use Storage in Kubernetes?**

By default, data inside a container is ephemeral. If a Pod is deleted or restarted, its data is lost. Kubernetes provides **Volumes** to solve this problem by enabling:
- Persistent storage for stateful applications (e.g., databases).
- Shared storage between containers in the same Pod.
- Integration with external storage systems (e.g., NFS, cloud storage).

---

## **What is a Volume?**

A **Volume** in Kubernetes is a directory accessible to containers within a Pod. Unlike container storage, Volumes:
- Persist data as long as the Pod exists.
- Can be backed by various storage backends (e.g., hostPath, NFS, Persistent Volumes).

### **Types of Volumes**
1. **emptyDir**: Temporary storage that exists as long as the Pod is running.
2. **hostPath**: Maps a directory on the host node to the Pod.
3. **PersistentVolume (PV)**: A cluster-wide storage resource provisioned by an admin.
4. **PersistentVolumeClaim (PVC)**: A request for storage by a user, bound to a PV.
5. **ConfigMap/Secret**: Mounts configuration or sensitive data as files.
6. **Cloud Provider Volumes**: Integrates with cloud storage like AWS EBS, GCP Persistent Disks, or Azure Disks.

---

## **Persistent Volumes (PV) and Persistent Volume Claims (PVC)**

### Persistent Volume (PV)
A PV is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using StorageClasses.

### Persistent Volume Claim (PVC)
A PVC is a request for storage by a user. It specifies the size and access mode needed. Once bound, a PVC can be used by Pods.

---

## **Creating Persistent Storage**

### Example: Persistent Volume (PV)

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

### Example: Persistent Volume Claim (PVC)

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

### Example: Using PVC in a Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
    - name: app-container
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage-volume
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: example-pvc
```

---

## **Activities**

### Activity #1: Create an emptyDir Volume
1. Create a Pod that uses an `emptyDir` volume to share data between two containers.
2. Example YAML snippet for `emptyDir`:

```
volumes:
  - name: shared-data
    emptyDir: {}
```

3. Verify that both containers can write to and read from the shared directory.

---

### Activity #2: Create PV and PVC for Persistent Storage
1. Create a Persistent Volume using the provided YAML example above.
2. Create a Persistent Volume Claim and bind it to your PV.
3. Create a Pod that mounts the PVC and writes data to it.
4. Verify that the data persists even if the Pod is deleted and recreated.

---

### Activity #3 (Optional): Use Cloud Storage
1. If you are using a cloud provider, create a PV backed by AWS EBS, GCP PD, or Azure Disk.
2. Bind it to a PVC and use it in your application.

---

## **Best Practices**

- Use **StorageClasses** for dynamic provisioning of Persistent Volumes.
- Choose appropriate access modes (`ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`) based on your application’s needs.
- Use cloud-native storage solutions for production workloads to ensure scalability and reliability.
- Regularly monitor and clean up unused PVCs and PVs to avoid resource wastage.

---

## **Additional References**

- Official Documentation on [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/) and [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- Blog Post on *"Understanding Kubernetes Storage"* [Link](https://www.cncf.io/blog/understanding-kubernetes-storage/)
- Video Tutorial on *"Kubernetes Storage Explained"* [YouTube Link](https://www.youtube.com/watch?v=JIbIYCM48to)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
