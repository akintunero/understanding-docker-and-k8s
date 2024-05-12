
# Day 7: Services

Welcome to Day 7 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Kubernetes Services**, which enable communication between Pods and expose applications to other Pods or external clients.

---

## **What is a Kubernetes Service?**

A **Service** is an abstraction in Kubernetes that defines a logical set of Pods and provides a stable endpoint (IP address or DNS name) for accessing them. Since Pods are ephemeral and can be replaced, their IP addresses may change. Services ensure consistent communication with Pods, even as they are created or destroyed.

### **Key Features of Services**
- Provides a stable IP address or DNS name for accessing Pods.
- Automatically load-balances traffic across the selected Pods.
- Supports different types of communication (internal and external).

---

## **Types of Services**

### **1. ClusterIP (Default)**
- Exposes the Service within the cluster.
- Accessible only from other Pods in the cluster.
- Use case: Internal communication between microservices.

### **2. NodePort**
- Exposes the Service on a static port on each node in the cluster.
- Accessible externally via `<NodeIP>:<NodePort>`.
- Use case: Simple external access for testing purposes.

### **3. LoadBalancer**
- Exposes the Service externally using a cloud provider’s load balancer.
- Use case: Production-grade external access.

### **4. ExternalName**
- Maps a Service to an external DNS name.
- Use case: Redirect traffic to an external service (e.g., a database hosted outside the cluster).

---

## **Creating a Service**

Here’s an example YAML file for creating a Service to expose an NGINX Deployment:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### Apply the Service:

```
kubectl apply -f nginx-service.yaml
```

### Verify the Service:

```
kubectl get services
kubectl describe service nginx-service
```

---

## **Accessing a Service**

1. **For `ClusterIP`**:
    - Access the Service from within the cluster using its DNS name or IP address.
    - Example command from another Pod:
      ```
      curl http://nginx-service
      ```

2. **For `NodePort`**:
    - Modify the YAML file to set `type: NodePort`.
    - Access the Service externally using `<NodeIP>:<NodePort>`.

3. **For `LoadBalancer`**:
    - Modify the YAML file to set `type: LoadBalancer`.
    - The cloud provider assigns an external IP address to access the Service.

---

## **Activities**

### Activity #1: Create a ClusterIP Service
1. Use the provided YAML file to create a ClusterIP Service for your NGINX Deployment from Day 6.
2. Verify that the Service is running using `kubectl get services`.
3. Access the Service from another Pod using `curl`.

### Activity #2: Create a NodePort Service
1. Modify your YAML file to create a NodePort Service by setting `type: NodePort`.
2. Apply the new configuration using `kubectl apply`.
3. Access your application externally using `<NodeIP>:<NodePort>`.

### Activity #3: Experiment with LoadBalancer (Optional)
1. If you are using a cloud provider like AWS, GCP, or Azure, create a LoadBalancer Service by setting `type: LoadBalancer`.
2. Verify that an external IP is assigned using `kubectl get services`.
3. Access your application via the external IP.

---

## **Additional References**

- Official Documentation: [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- Blog Post: *"Understanding Kubernetes Services"* by CNCF.
- Video Tutorial: *"Kubernetes Services Explained"* [YouTube Link](https://www.youtube.com/watch?v=5OkG6dXzZcA)
- Interactive Lab: Practice creating Services on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
