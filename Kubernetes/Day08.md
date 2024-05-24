
# Day 8: ConfigMaps and Secrets

Welcome to Day 8 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **ConfigMaps** and **Secrets**, two essential Kubernetes resources for managing configuration data and sensitive information in a secure and modular way.

---

## **What is a ConfigMap?**

A **ConfigMap** is a Kubernetes API object used to store non-sensitive configuration data as key-value pairs. It helps decouple configuration details from application code, making applications more portable and easier to manage.

### **Key Features of ConfigMaps**
- Stores non-sensitive data like environment variables, file paths, or URLs.
- Can be consumed by Pods as:
    - Environment variables.
    - Command-line arguments.
    - Mounted files.
- Promotes modularity by separating configuration from container images.

### **Example: ConfigMap YAML**

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-configmap
data:
  database_url: "mongodb://db.example.com:27017"
  feature_flag: "true"
```

---

## **What is a Secret?**

A **Secret** is a Kubernetes API object used to store sensitive information like passwords, API keys, or tokens. Secrets are similar to ConfigMaps but provide an additional layer of security by encoding the data in base64.

### **Key Features of Secrets**
- Stores sensitive data securely.
- Can be consumed by Pods as:
    - Environment variables.
    - Mounted files.
- Supports encryption at rest (must be enabled explicitly).

### **Example: Secret YAML**

```
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  username: YWRtaW4= # base64 encoded 'admin'
  password: MWYyZDFlMmU2N2Rm # base64 encoded '1f2d1e2e67df'
```

---

## **Differences Between ConfigMaps and Secrets**

| Feature                  | ConfigMap                       | Secret                           |
|--------------------------|----------------------------------|----------------------------------|
| Purpose                  | Non-sensitive configuration     | Sensitive information           |
| Data Format              | Plaintext key-value pairs       | Base64-encoded key-value pairs  |
| Security                 | No encryption                  | Supports encryption at rest     |
| Use Cases                | Environment variables, configs  | Passwords, API keys, tokens     |

---

## **Using ConfigMaps and Secrets in Pods**

### **1. Injecting as Environment Variables**
**Example Pod using a ConfigMap:**

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: app-container
      image: nginx
      env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: example-configmap
              key: database_url
```

**Example Pod using a Secret:**

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: app-container
      image: nginx
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: example-secret
              key: username
```

### **2. Mounting as Volumes**
ConfigMap or Secret data can also be mounted as files in a container.

**Example for mounting a ConfigMap as a volume:**

```
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
    - name: app-container
      image: nginx
      volumeMounts:
        - mountPath: "/etc/config"
          name: config-volume
  volumes:
    - name: config-volume
      configMap:
        name: example-configmap
```

---

## **Activities**

### Activity #1: Create and Use a ConfigMap
1. Create a ConfigMap using the following YAML file:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  app_name: "MyApp"
  log_level: "DEBUG"
```

Apply it using `kubectl apply -f`.

2. Create a Pod that uses this ConfigMap as environment variables.

3. Verify the environment variables inside the Pod using `kubectl exec` to access the container.

### Activity #2: Create and Use a Secret
1. Create a Secret using the following YAML file:

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4= # base64 for 'admin'
  password: cGFzc3dvcmQ= # base64 for 'password'
```

Apply it using `kubectl apply -f`.

2. Create a Pod that uses this Secret as environment variables.

3. Verify the secret values inside the Pod using `kubectl exec`.

### Activity #3 (Optional): Mount as Volumes
Modify your Pod definitions to mount the ConfigMap or Secret as volumes and inspect the mounted files inside the container.

---

## **Best Practices**

- Use **Secrets** for sensitive information only; avoid storing them in plaintext.
- Enable **encryption at rest** for Secrets to enhance security.
- Limit access to ConfigMaps and Secrets using RBAC (Role-Based Access Control).
- Mark frequently used ConfigMaps and Secrets as immutable to prevent accidental changes.
- Regularly rotate Secrets to minimize risks from potential breaches.

---

## **Additional References**

- Official Documentation on [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) and [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- Blog Post on *"Managing Kubernetes ConfigMaps & Secrets"* [Link](https://www.getambassador.io/blog/kubernetes-configurations-secrets-configmaps)
- Video Tutorial on *"Kubernetes ConfigMaps and Secrets Explained"* [YouTube Link](https://www.youtube.com/watch?v=6df7O7mJXzI)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
