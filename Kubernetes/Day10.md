
# Day 10: Networking Basics

Welcome to Day 10 of the Kubernetes 20-Day Learning Challenge! Today, we will explore the basics of Kubernetes networking, including how Pods communicate with each other, how Services handle traffic, and an introduction to Network Policies for securing communication.

---

## **Kubernetes Networking Overview**

Kubernetes networking is a critical component that ensures communication between:
1. **Pods** (Pod-to-Pod communication).
2. **Services** (Pod-to-Service communication).
3. **External clients** (Internet-to-Service communication).

### **Key Features of Kubernetes Networking**
- Each Pod gets a unique cluster-wide IP address.
- Pods can communicate with each other directly without requiring Network Address Translation (NAT).
- Containers within the same Pod share a network namespace and can communicate over `localhost`.
- Services provide stable endpoints for Pods, enabling load balancing and external access.

---

## **Types of Networking in Kubernetes**

### **1. Pod-to-Pod Networking**
- Each Pod has a unique IP address that allows it to communicate with other Pods in the cluster.
- Communication between Pods on the same Node is handled via a virtual ethernet bridge.
- For Pods on different Nodes, traffic is routed through the cluster network.

### **2. Pod-to-Service Networking**
- Services provide a stable IP address and DNS name for accessing a group of Pods.
- Kubernetes uses kube-proxy to route traffic from Services to the appropriate Pods.

### **3. Internet-to-Service Networking**
- External traffic can be routed into the cluster using Service types like `NodePort` or `LoadBalancer`.
- Ingress resources provide advanced routing capabilities for HTTP/HTTPS traffic.

---

## **Network Policies**

**Network Policies** are used to control traffic flow at the Pod level. They define which Pods can communicate with each other and with external endpoints.

### Components of a Network Policy:
1. **Ingress Rules**: Control incoming traffic to Pods.
2. **Egress Rules**: Control outgoing traffic from Pods.
3. **Selectors**: Define which Pods the policy applies to using labels.

### Example: Denying All Traffic by Default

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### Example: Allowing Specific Ingress Traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: my-app
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 80
```

---

## **Activities**

### Activity #1: Explore Pod-to-Pod Communication
1. Deploy two Pods using simple YAML files.
2. Use `kubectl exec` to access one Pod and ping the other Pod using its IP address.
3. Verify that communication works without additional configuration.

### Activity #2: Create a Service for Pod Communication
1. Create a Deployment for an NGINX application.
2. Expose it as a `ClusterIP` Service.
3. Access the Service from another Pod using its DNS name or IP address.

### Activity #3: Apply a Network Policy (Optional)
1. Create a Network Policy to allow only specific Pods to communicate with your NGINX application.
2. Test the policy by trying to access the application from allowed and disallowed Pods.

---

## **Best Practices**

- Use Network Policies to restrict unnecessary communication between Pods.
- Monitor and log network traffic to identify potential issues or security risks.
- Use Ingress resources for advanced routing and TLS termination when exposing applications externally.

---

## **Additional References**

- [Kubernetes Networking Overview](https://kubernetes.io/docs/concepts/services-networking/)
- [Understanding Kubernetes Network Policies](https://deploy.equinix.com/blog/understanding-kubernetes-network-policies/)
- [Kubernetes Networking Basics](https://spacelift.io/blog/kubernetes-networking)
- [Mastering Kubernetes Networking](https://www.getambassador.io/blog/kubernetes-networking-guide-top-engineers)

---
