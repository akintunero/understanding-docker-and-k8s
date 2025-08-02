
# Day 15: Network Policies

Welcome to Day 15 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Network Policies**, which allow you to control network traffic flow between Pods, namespaces, and external endpoints in your Kubernetes cluster.

---

## **What is a Network Policy?**

A **Network Policy** is a Kubernetes resource that defines how Pods are allowed to communicate with each other and with other network endpoints. By default, Kubernetes follows an "allow-any-any" model, meaning all Pods can communicate freely. Network Policies enable fine-grained control over this communication.

### **Key Features of Network Policies**
1. Control both **Ingress** (incoming) and **Egress** (outgoing) traffic.
2. Apply rules to Pods based on labels.
3. Define traffic rules based on:
    - Pod selectors
    - Namespace selectors
    - IP blocks

### **Why Use Network Policies?**
- Improve security by isolating workloads.
- Prevent unauthorized communication between Pods.
- Implement multi-tenancy by restricting cross-namespace traffic.

---

## **How Network Policies Work**

1. **Default Behavior**: Without a Network Policy, all traffic is allowed between Pods.
2. **Selective Isolation**: Once a Network Policy is applied to a Pod, only traffic explicitly allowed by the policy is permitted.
3. **Policy Types**:
    - **Ingress**: Controls incoming traffic to Pods.
    - **Egress**: Controls outgoing traffic from Pods.

---

## **Creating a Network Policy**

Here’s an example YAML file for a Network Policy that allows only specific Pods to access a database Pod:

### Example: Allow Specific Ingress Traffic

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific-ingress
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 3306
```

### Explanation:
1. **podSelector**: Targets Pods with the label `role=db`.
2. **policyTypes**: Specifies that this policy applies to incoming (Ingress) traffic.
3. **ingress.from**: Allows traffic only from Pods with the label `role=frontend`.
4. **ports**: Restricts access to TCP port `3306` (commonly used by MySQL).

---

## **Testing a Network Policy**

1. Apply the policy using:

```
kubectl apply -f network_policy.yaml
```

2. Verify the policy is active:

```
kubectl get networkpolicy allow-specific-ingress -n default
```

3. Test communication between Pods using tools like `curl` or `ping`.

---

## **Activities**

### Activity #1: Create a Deny-All Policy
1. Create a Network Policy that denies all incoming and outgoing traffic for a specific Pod.
2. Example YAML snippet:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-traffic
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
```

3. Apply the policy and test that no communication is allowed.

---

### Activity #2: Allow Namespace-Specific Traffic (Optional)
1. Create a policy that allows traffic only from Pods in the same namespace.
2. Example YAML snippet:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: my-app
  ingress:
  - from:
    - namespaceSelector: {}
```

3. Apply the policy and verify that cross-namespace communication is blocked.

---

### Activity #3 (Optional): Allow Traffic from Specific IP Blocks
1. Create a policy that allows traffic only from specific IP ranges.
2. Example YAML snippet:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ip-blocks
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: my-app
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.0.0/16
        except:
        - 192.168.1.0/24
    ports:
    - protocol: TCP
      port: 80
```

3. Test access by sending requests from allowed and disallowed IPs.

---

## **Best Practices**

- Always start with a deny-all policy and explicitly allow required traffic.
- Use precise selectors (Pod labels, namespaces, IP blocks) to avoid unintended access.
- Regularly review and update policies as your cluster evolves.
- Monitor network activity to ensure policies are enforced correctly.

---

## **Additional References**

- Official Documentation on [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- Blog Post on *"Kubernetes Network Policies Explained"* [Link](https://spacelift.io/blog/kubernetes-network-policy)
- Video Tutorial on *"Kubernetes Network Policies"* [YouTube Link](https://www.youtube.com/watch?v=V3XoYy5J9iM)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
