# Day 18: Security and RBAC

Welcome to Day 18 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Kubernetes Security** with a focus on **Role-Based Access Control (RBAC)**, which is a critical mechanism for managing access to resources in your cluster.

---

## **What is Kubernetes RBAC?**

**Role-Based Access Control (RBAC)** is a Kubernetes feature that allows you to define and enforce access control policies. It ensures that users, groups, or service accounts have only the permissions they need to perform their tasks.

### **Key Concepts of RBAC**
1. **Roles and ClusterRoles**:
    - A **Role** defines permissions within a specific namespace.
    - A **ClusterRole** defines permissions across the entire cluster.

2. **RoleBindings and ClusterRoleBindings**:
    - A **RoleBinding** assigns a Role to a user, group, or service account within a namespace.
    - A **ClusterRoleBinding** assigns a ClusterRole across all namespaces.

3. **Principle of Least Privilege**:
    - Grant only the minimum permissions required for users or workloads to perform their tasks.

---

## **Why Use RBAC?**

RBAC is essential for:
- Securing access to sensitive resources in your cluster.
- Preventing unauthorized actions by limiting privileges.
- Auditing and managing access control policies effectively.

---

## **Creating RBAC Policies**

### Example: Creating a Role
The following YAML file creates a Role that allows reading Pods in the `default` namespace:

````
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
namespace: default
name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
 ````

### Example: Creating a RoleBinding
The following YAML file binds the `pod-reader` Role to a user named `jane`:

````
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: read-pods
namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: ""
  roleRef:
  kind: Role
  name: pod-reader
  apiGroup: ""
````

### Apply the Role and RoleBinding:

````
kubectl apply -f role.yaml
kubectl apply -f role_binding.yaml
````

---

## **Best Practices for RBAC**

1. **Follow the Principle of Least Privilege**:
    - Grant only the permissions required for each role.
    - Avoid assigning broad ClusterRoles unless absolutely necessary.

2. **Use Namespaces for Isolation**:
    - Scope Roles and RoleBindings to specific namespaces where possible.
    - Use namespaces to separate environments (e.g., dev, staging, production).

3. **Regularly Audit Permissions**:
    - Review and update RBAC policies periodically to ensure they align with current requirements.
    - Remove unused roles and bindings.

4. **Use Service Accounts for Applications**:
    - Assign specific service accounts to applications instead of using default accounts.
    - Bind only necessary Roles or ClusterRoles to service accounts.

5. **Enable Logging and Monitoring**:
    - Use Kubernetes audit logs to track who accessed what resources and when.
    - Monitor changes to RBAC policies for unauthorized modifications.

---

## **Activities**

### Activity #1: Create an RBAC Policy for Pod Readers
1. Create a Role that allows reading Pods in the `default` namespace.
2. Bind this Role to a user or service account using a RoleBinding.
3. Test access by attempting to list Pods as the assigned user.

### Activity #2: Create a Cluster-Wide Policy (Optional)
1. Create a ClusterRole that allows listing nodes across the cluster.
2. Bind this ClusterRole to a user using a ClusterRoleBinding.
3. Verify that the user can list nodes but cannot perform other actions.

### Activity #3 (Optional): Audit Existing RBAC Policies
1. List all Roles and ClusterRoles in your cluster using:

````
kubectl get roles --all-namespaces
kubectl get clusterroles
````

2. Identify any overly permissive roles and update them to follow the principle of least privilege.

---

## **Additional Security Practices**

In addition to RBAC, consider implementing these security measures:

1. **Secure API Server Access**:
- Enable TLS encryption for all API traffic.
- Limit access to trusted networks only.

2. **Protect etcd**:
- Use encryption at rest for etcd data.
- Restrict access to etcd with strong credentials and firewalls.

3. **Enable Network Policies**:
- Use Network Policies to restrict Pod-to-Pod communication.
- Block unnecessary traffic by default and explicitly allow required communication.

4. **Manage Secrets Securely**:
- Encrypt secrets stored in etcd.
- Limit access to secrets using RBAC policies.

5. **Regularly Update Kubernetes Components**:
- Keep your cluster up-to-date with the latest patches and security fixes.

---

## **Additional References**

- Official Documentation on [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- Blog Post on *"Kubernetes Security Best Practices"* [Link](https://www.getambassador.io/blog/kubernetes-security-best-practices-secure-environment)
- Video Tutorial on *"Kubernetes RBAC Explained"* [YouTube Link](https://www.youtube.com/watch?v=VZTF8lBfLwE)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
