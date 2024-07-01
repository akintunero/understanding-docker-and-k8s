
# Day 11: StatefulSets

Welcome to Day 11 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **StatefulSets**, a Kubernetes resource designed for managing stateful applications that require persistent storage and stable network identities.

---

## **What is a StatefulSet?**

A **StatefulSet** is a Kubernetes workload API object used to manage stateful applications. Unlike Deployments, which are suitable for stateless applications, StatefulSets are designed for applications that require:
- Stable, unique network identities.
- Persistent storage across Pod restarts.
- Ordered deployment, scaling, and termination.

### **Key Features of StatefulSets**
1. **Stable Network Identity**:
    - Each Pod in a StatefulSet gets a unique, predictable name (e.g., `pod-name-0`, `pod-name-1`).
    - These names remain consistent even if Pods are rescheduled.

2. **Persistent Storage**:
    - Each Pod can have its own PersistentVolumeClaim (PVC) for data persistence.
    - The PVCs are not shared between Pods and remain bound to their respective Pods.

3. **Ordered Operations**:
    - Pods are created, updated, and deleted in a specific order (e.g., Pod `0` is created before Pod `1`).

---

## **When to Use StatefulSets**

StatefulSets are ideal for applications that require:
- Persistent data storage (e.g., databases like MySQL, PostgreSQL).
- Unique Pod identities (e.g., Zookeeper, Kafka).
- Ordered operations (e.g., leader election in distributed systems).

For stateless applications, use Deployments instead.

---

## **Creating a StatefulSet**

Here’s an example YAML file for creating a simple StatefulSet with NGINX:

### Example: StatefulSet with Persistent Storage

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: www
              mountPath: "/usr/share/nginx/html"
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

### Key Components of the YAML File:
1. **serviceName**: A headless Service is required to manage network identities for Pods.
2. **volumeClaimTemplates**: Defines the PersistentVolumeClaims for each Pod.
3. **replicas**: Specifies the number of Pods in the StatefulSet.

---

## **Managing StatefulSets**

### Create the StatefulSet:

```
kubectl apply -f statefulset.yaml
```

### Check the StatefulSet and its Pods:

```
kubectl get statefulsets
kubectl get pods -l app=nginx
```

### Delete the StatefulSet (PVCs are retained):

```
kubectl delete statefulset nginx-statefulset
```

---

## **Activities**

### Activity #1: Create a Simple StatefulSet
1. Use the provided YAML file to create an NGINX StatefulSet with persistent storage.
2. Verify that each Pod has a unique name (`nginx-statefulset-0`, `nginx-statefulset-1`, etc.).

### Activity #2: Inspect Persistent Volumes
1. List all PersistentVolumeClaims (PVCs) created by the StatefulSet using:

```
kubectl get pvc
```

2. Verify that each PVC is bound to its respective Pod.

### Activity #3 (Optional): Scale the StatefulSet
1. Scale the StatefulSet to add more replicas using:

```
kubectl scale statefulset nginx-statefulset --replicas=5
```

2. Verify that new Pods are created in order (`nginx-statefulset-3`, `nginx-statefulset-4`).

---

## **Best Practices**

- Use **StatefulSets** only when your application requires stable identities or persistent storage.
- Use a headless Service (`clusterIP: None`) with StatefulSets to enable direct communication between Pods.
- Avoid scaling down below the minimum number of replicas required by your application’s architecture.
- Monitor PVC usage and clean up unused volumes when deleting StatefulSets.

---

## **Additional References**

- Official Documentation on [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- Blog Post on *"Understanding Kubernetes StatefulSets"* [Link](https://www.cncf.io/blog/kubernetes-statefulsets/)
- Video Tutorial on *"Kubernetes StatefulSets Explained"* [YouTube Link](https://www.youtube.com/watch?v=E7TxEZx8KXU)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
