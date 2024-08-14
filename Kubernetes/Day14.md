
# Day 14: Ingress Controllers

Welcome to Day 14 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Ingress Controllers**, which are used to manage external access to services in a Kubernetes cluster, typically over HTTP and HTTPS.

---

## **What is an Ingress?**

An **Ingress** is a Kubernetes API object that provides HTTP and HTTPS routing to Services within a cluster. It acts as a reverse proxy, allowing you to expose multiple Services through a single external IP address or domain name.

### **Key Features of Ingress**
1. Routes traffic based on hostnames or paths (e.g., `example.com/api`).
2. Supports SSL/TLS termination for secure communication.
3. Reduces the need for multiple LoadBalancer or NodePort Services.

---

## **What is an Ingress Controller?**

An **Ingress Controller** is a component that implements the Ingress API and processes Ingress resources. It is responsible for routing traffic from outside the cluster to the appropriate Services based on the rules defined in the Ingress resource.

### Popular Ingress Controllers:
- NGINX Ingress Controller
- Traefik
- HAProxy
- AWS ALB Ingress Controller (for AWS environments)

---

## **How Ingress Works**

1. A user creates an Ingress resource to define routing rules.
2. The Ingress Controller watches for changes to Ingress resources and updates its configuration accordingly.
3. External traffic is routed through the Ingress Controller and forwarded to the appropriate Service.

---

## **Setting Up an Ingress Controller**

### Step 1: Install an Ingress Controller
For this example, we will use the NGINX Ingress Controller.

#### Install via Helm:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx
```

#### Verify Installation:

```
kubectl get pods -n default -l app.kubernetes.io/name=ingress-nginx
kubectl get svc -n default -l app.kubernetes.io/name=ingress-nginx
```

---

## **Creating an Ingress Resource**

Here’s an example YAML file for creating an Ingress resource:

### Example: Simple Ingress Resource

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

### Key Components of the YAML File:
1. **rules**: Defines routing rules based on hostnames and paths.
2. **backend**: Specifies the Service and port to forward traffic to.

---

## **Enabling HTTPS with TLS**

To enable HTTPS, you need to provide a TLS certificate and key.

### Example: TLS Configuration in an Ingress Resource

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
    - hosts:
        - example.com
      secretName: tls-secret # Reference to the TLS secret
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

### Create a TLS Secret:

```
kubectl create secret tls tls-secret --cert=path/to/tls.crt --key=path/to/tls.key
```

---

## **Activities**

### Activity #1: Deploy an NGINX Service with an Ingress Resource
1. Create a Deployment and Service for NGINX.
2. Create an Ingress resource to route traffic to your NGINX Service.
3. Test access by visiting the hostname specified in your `Ingress` resource.

### Activity #2 (Optional): Enable HTTPS with TLS
1. Generate a self-signed certificate or use a real certificate.
2. Create a TLS Secret using `kubectl create secret`.
3. Update your Ingress resource to include TLS configuration.
4. Test access using `https://<hostname>`.

### Activity #3 (Optional): Use Path-Based Routing
1. Create two Services (e.g., `service-a` and `service-b`).
2. Update your Ingress resource to route `/app-a` to `service-a` and `/app-b` to `service-b`.
3. Test access by visiting `/app-a` and `/app-b`.

---

## **Best Practices**

- Use DNS names instead of IP addresses for better scalability and readability.
- Always enable HTTPS with valid TLS certificates for secure communication.
- Monitor your Ingress resources and controllers for performance and errors.
- Use annotations supported by your specific Ingress Controller for advanced configurations like rate limiting or custom headers.

---

## **Additional References**

- Official Documentation on [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- NGINX Ingress Controller Documentation: [NGINX GitHub](https://github.com/kubernetes/ingress-nginx)
- Blog Post on *"Kubernetes Networking with Ingress Controllers"* [Link](https://www.cncf.io/blog/kubernetes-ingress/)
- Video Tutorial on *"Kubernetes Ingress Explained"* [YouTube Link](https://www.youtube.com/watch?v=7xngnjfIlK4)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
