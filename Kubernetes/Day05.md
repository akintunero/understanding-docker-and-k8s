
# Day 5: Basic kubectl Commands

Welcome to Day 5 of the Kubernetes 20-Day Learning Challenge! Today, we will focus on mastering essential `kubectl` commands, which are used to interact with and manage Kubernetes clusters. These commands are critical for deploying applications, inspecting resources, and troubleshooting.

---

## **What is kubectl?**

`kubectl` is the command-line tool for interacting with Kubernetes clusters. It allows you to:
- Deploy and manage applications.
- Inspect and modify cluster resources.
- Debug and troubleshoot issues.

### **kubectl Syntax**
The general syntax for `kubectl` commands is:

```
kubectl [command] [TYPE] [NAME] [flags]
```

- **command**: The action to perform (e.g., `get`, `create`, `delete`).
- **TYPE**: The resource type (e.g., `pod`, `service`, `deployment`).
- **NAME**: The name of the resource (optional).
- **flags**: Additional options (e.g., `--namespace`, `--output`).

---

## **Essential kubectl Commands**

### **1. Viewing Cluster Information**
- Check cluster status:

  ```
  kubectl cluster-info
  ```
- List all nodes in the cluster:

  ```
  kubectl get nodes
  ```

### **2. Managing Pods**
- List all Pods:

  ```
  kubectl get pods
  ```
- View details of a specific Pod:

  ```
  kubectl describe pod <pod-name>
  ```
- View logs from a Pod’s container:

  ```
  kubectl logs <pod-name>
  ```
- Execute a command inside a running Pod:

  ```
  kubectl exec -it <pod-name> -- /bin/bash
  ```

### **3. Managing Deployments**
- List all Deployments:

  ```
  kubectl get deployments
  ```
- Scale a Deployment (e.g., increase replicas):

  ```
  kubectl scale deployment <deployment-name> --replicas=3
  ```

### **4. Creating and Deleting Resources**
- Apply a resource definition from a YAML file:

  ```
  kubectl apply -f <file-name>.yaml
  ```
- Delete a resource (e.g., Pod, Deployment):

  ```
  kubectl delete <resource-type> <resource-name>
  ```

### **5. Namespace Management**
- List all namespaces:

  ```
  kubectl get namespaces
  ```
- Switch to a specific namespace:

  ```
  kubectl config set-context --current --namespace=<namespace-name>
  ```

### **6. Advanced Output Options**
- Display resources in wide format (more details):

  ```
  kubectl get pods -o wide
  ```
- Output resource details in YAML or JSON format:

  ```
  kubectl get pod <pod-name> -o yaml
  ```

---

## **Activities**

### Activity #1: Explore Your Cluster
1. Use `kubectl cluster-info` and `kubectl get nodes` to inspect your cluster.
2. Verify that your local Kubernetes setup is running correctly.

### Activity #2: Create and Manage Pods
1. Create a simple Pod using the following YAML file:

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-nginx-pod
   spec:
     containers:
       - name: nginx-container
         image: nginx:latest
         ports:
           - containerPort: 80
   ```

   Apply it using:

   ```
   kubectl apply -f pod.yaml
   ```

2. List all Pods using `kubectl get pods`.

3. Describe your Pod using `kubectl describe pod my-nginx-pod`.

4. Delete the Pod using:

   ```
   kubectl delete pod my-nginx-pod
   ```

### Activity #3: Scale a Deployment (Optional)
1. Create a Deployment using the following YAML file:

   ```
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
           - name: nginx-container
             image: nginx:latest
             ports:
               - containerPort: 80
   ```

2. Apply the Deployment using `kubectl apply -f deployment.yaml`.

3. Scale the Deployment to three replicas using:

   ```
   kubectl scale deployment nginx-deployment --replicas=3
   ```

4. Verify the updated replicas using `kubectl get pods`.

---

## **Additional References**

- Official Documentation: [kubectl Overview](https://kubernetes.io/docs/reference/kubectl/)
- Cheat Sheet: [kubectl Command Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- Video Tutorial: *"Mastering kubectl Commands"* [YouTube Link](https://www.youtube.com/watch?v=ZtqBQ68cfJc)
- Interactive Lab: Practice commands on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
