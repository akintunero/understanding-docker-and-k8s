
# Day 6: Deployments

Welcome to Day 6 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Kubernetes Deployments**, a key resource for managing the lifecycle of applications, including creation, scaling, and updates.

---

## **What is a Kubernetes Deployment?**

A **Deployment** is a Kubernetes resource object that provides declarative updates to applications. It allows you to define the desired state of your application, and Kubernetes ensures that the actual state matches this desired state.

### **Key Features of Deployments**
- **Declarative Updates**: Specify the desired state (e.g., number of replicas, container image) in a YAML file, and Kubernetes handles the rest.
- **Self-Healing**: Automatically replaces failed Pods to maintain the desired number of replicas.
- **Rolling Updates**: Gradually update Pods with minimal downtime.
- **Scaling**: Easily scale applications up or down by changing the number of replicas.
- **Rollback**: Revert to a previous version if something goes wrong.

---

## **How Deployments Work**

1. A Deployment creates a **ReplicaSet**, which in turn manages the Pods.
2. The ReplicaSet ensures that the specified number of Pods are running at all times.
3. If a Pod fails or is deleted, the ReplicaSet automatically replaces it.

For example:
- You define a Deployment with 3 replicas of an application.
- Kubernetes ensures that exactly 3 Pods are running at all times. If one fails, it is replaced automatically.

---

## **Creating a Deployment**

Here’s an example YAML file for creating an NGINX Deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
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
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### Apply the Deployment:

```
kubectl apply -f nginx-deployment.yaml
```

### Verify the Deployment:

```
kubectl get deployments
kubectl get pods
```

---

## **Scaling a Deployment**

You can scale your application by increasing or decreasing the number of replicas.

### Scale Command:

```
kubectl scale deployment nginx-deployment --replicas=5
```

Verify scaling:

```
kubectl get pods
```

---

## **Updating a Deployment**

To update an application (e.g., change the container image):
1. Edit the Deployment YAML file or use `kubectl edit deployment`.
2. Update the `image` field (e.g., `nginx:1.16.0`).
3. Apply changes using `kubectl apply`.

Kubernetes performs a rolling update to ensure no downtime during the update process.

---

## **Rollback a Deployment**

If an update causes issues, roll back to a previous version using:

```
kubectl rollout undo deployment nginx-deployment
```

Check rollout status:

```
kubectl rollout status deployment nginx-deployment
```

---

## **Activities**

### Activity #1: Create Your First Deployment
1. Use the YAML file provided above to create an NGINX Deployment.
2. Verify that 3 Pods are running using `kubectl get pods`.

### Activity #2: Scale Your Deployment
1. Scale your Deployment to 5 replicas using `kubectl scale`.
2. Verify that Kubernetes has created additional Pods.

### Activity #3: Update Your Deployment (Optional)
1. Update the NGINX image to version `nginx:1.16.0`.
2. Observe how Kubernetes performs a rolling update using `kubectl get pods`.

### Activity #4: Rollback (Optional)
1. Roll back your Deployment to its previous version using `kubectl rollout undo`.
2. Verify that Kubernetes has reverted to the earlier configuration.

---

## **Additional References**

- Official Documentation: [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- Video Tutorial: *"Kubernetes Deployments Explained"* [YouTube Link](https://www.youtube.com/watch?v=JfJY4fDq8Bc)
- Interactive Lab: Practice creating Deployments on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
