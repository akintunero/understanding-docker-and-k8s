# Lab 02: Deployments and ReplicaSets

## 🎯 Learning Objectives
- Create and manage Kubernetes Deployments
- Understand ReplicaSets and their relationship to Deployments
- Implement rolling updates and rollbacks
- Configure deployment strategies
- Practice scaling and resource management

## ⏱️ Estimated Time: 60 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of pods (Lab 01)

## 🚀 Lab Exercises

### Exercise 1: Creating Your First Deployment

1. **Create a simple deployment:**
   ```bash
   # Create deployment YAML
   cat << EOF > simple-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
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
           image: nginx:alpine
           ports:
           - containerPort: 80
   EOF
   
   # Apply the deployment
   kubectl apply -f simple-deployment.yaml
   ```

2. **Verify deployment status:**
   ```bash
   # Check deployment status
   kubectl get deployments
   kubectl get pods -l app=nginx
   
   # Get detailed deployment information
   kubectl describe deployment nginx-deployment
   
   # Check ReplicaSet
   kubectl get replicasets
   ```

### Exercise 2: Scaling Deployments

1. **Scale the deployment:**
   ```bash
   # Scale up to 5 replicas
   kubectl scale deployment nginx-deployment --replicas=5
   
   # Verify scaling
   kubectl get pods -l app=nginx
   kubectl get deployment nginx-deployment
   
   # Scale down to 2 replicas
   kubectl scale deployment nginx-deployment --replicas=2
   
   # Check final state
   kubectl get pods -l app=nginx
   ```

2. **Scale using YAML:**
   ```bash
   # Update the deployment
   kubectl patch deployment nginx-deployment -p '{"spec":{"replicas":4}}'
   
   # Or edit directly
   kubectl edit deployment nginx-deployment
   ```

### Exercise 3: Rolling Updates

1. **Update the deployment:**
   ```bash
   # Update image to a newer version
   kubectl set image deployment/nginx-deployment nginx=nginx:1.21-alpine
   
   # Watch the rolling update
   kubectl rollout status deployment/nginx-deployment
   
   # Check deployment history
   kubectl rollout history deployment/nginx-deployment
   ```

2. **Update with YAML:**
   ```bash
   # Create updated deployment
   cat << EOF > updated-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
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
           image: nginx:1.21-alpine
           ports:
           - containerPort: 80
   EOF
   
   # Apply the update
   kubectl apply -f updated-deployment.yaml
   ```

### Exercise 4: Rollback Strategies

1. **Perform a rollback:**
   ```bash
   # Check rollout history
   kubectl rollout history deployment/nginx-deployment
   
   # Rollback to previous version
   kubectl rollout undo deployment/nginx-deployment
   
   # Watch rollback progress
   kubectl rollout status deployment/nginx-deployment
   
   # Check current image
   kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
   ```

2. **Rollback to specific revision:**
   ```bash
   # List revision history
   kubectl rollout history deployment/nginx-deployment --revision=1
   
   # Rollback to specific revision
   kubectl rollout undo deployment/nginx-deployment --to-revision=1
   ```

### Exercise 5: Deployment Strategies

1. **Create deployment with RollingUpdate strategy:**
   ```bash
   # Create deployment with rolling update strategy
   cat << EOF > rolling-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: rolling-app
     labels:
       app: rolling-app
   spec:
     replicas: 5
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxSurge: 2
         maxUnavailable: 1
     selector:
       matchLabels:
         app: rolling-app
     template:
       metadata:
         labels:
           app: rolling-app
       spec:
         containers:
         - name: app
           image: nginx:alpine
           ports:
           - containerPort: 80
   EOF
   
   kubectl apply -f rolling-deployment.yaml
   ```

2. **Create deployment with Recreate strategy:**
   ```bash
   # Create deployment with recreate strategy
   cat << EOF > recreate-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: recreate-app
     labels:
       app: recreate-app
   spec:
     replicas: 3
     strategy:
       type: Recreate
     selector:
       matchLabels:
         app: recreate-app
     template:
       metadata:
         labels:
           app: recreate-app
       spec:
         containers:
         - name: app
           image: nginx:alpine
           ports:
           - containerPort: 80
   EOF
   
   kubectl apply -f recreate-deployment.yaml
   ```

### Exercise 6: Resource Management

1. **Create deployment with resource limits:**
   ```bash
   # Create deployment with resource requests and limits
   cat << EOF > resource-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: resource-app
     labels:
       app: resource-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: resource-app
     template:
       metadata:
         labels:
           app: resource-app
       spec:
         containers:
         - name: app
           image: nginx:alpine
           ports:
           - containerPort: 80
           resources:
             requests:
               memory: "64Mi"
               cpu: "250m"
             limits:
               memory: "128Mi"
               cpu: "500m"
   EOF
   
   kubectl apply -f resource-deployment.yaml
   ```

2. **Monitor resource usage:**
   ```bash
   # Check resource usage
   kubectl top pods -l app=resource-app
   
   # Get detailed resource information
   kubectl describe pods -l app=resource-app
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Blue-Green Deployment

1. **Create blue deployment:**
   ```bash
   # Create blue deployment
   cat << EOF > blue-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: blue-app
     labels:
       app: blue-app
       version: blue
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: blue-app
         version: blue
     template:
       metadata:
         labels:
           app: blue-app
           version: blue
       spec:
         containers:
         - name: app
           image: nginx:1.20-alpine
           ports:
           - containerPort: 80
   EOF
   
   kubectl apply -f blue-deployment.yaml
   ```

2. **Create green deployment:**
   ```bash
   # Create green deployment
   cat << EOF > green-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: green-app
     labels:
       app: green-app
       version: green
   spec:
     replicas: 0
     selector:
       matchLabels:
         app: green-app
         version: green
     template:
       metadata:
         labels:
           app: green-app
           version: green
       spec:
         containers:
         - name: app
           image: nginx:1.21-alpine
           ports:
           - containerPort: 80
   EOF
   
   kubectl apply -f green-deployment.yaml
   ```

3. **Implement blue-green switch:**
   ```bash
   # Scale up green deployment
   kubectl scale deployment green-app --replicas=3
   
   # Wait for green to be ready
   kubectl rollout status deployment/green-app
   
   # Scale down blue deployment
   kubectl scale deployment blue-app --replicas=0
   
   # Verify switch
   kubectl get pods -l app=blue-app
   kubectl get pods -l app=green-app
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you create and manage Deployments?
- [ ] Do you understand the relationship between Deployments and ReplicaSets?
- [ ] Can you perform rolling updates and rollbacks?
- [ ] Can you configure deployment strategies?
- [ ] Do you understand resource management in Deployments?
- [ ] Can you implement blue-green deployments?

### Skills Demonstrated:
- ✅ Deployment creation and management
- ✅ ReplicaSet understanding
- ✅ Rolling update implementation
- ✅ Rollback strategies
- ✅ Resource management
- ✅ Advanced deployment patterns

## 🔍 Troubleshooting

### Common Issues:

1. **Deployment not scaling:**
   ```bash
   # Check deployment status
   kubectl describe deployment <deployment-name>
   
   # Check pod status
   kubectl get pods -l app=<app-label>
   
   # Check events
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

2. **Rollout stuck:**
   ```bash
   # Check rollout status
   kubectl rollout status deployment/<deployment-name>
   
   # Check deployment history
   kubectl rollout history deployment/<deployment-name>
   
   # Force rollback if needed
   kubectl rollout undo deployment/<deployment-name>
   ```

3. **Resource issues:**
   ```bash
   # Check resource usage
   kubectl top pods
   kubectl top nodes
   
   # Check resource limits
   kubectl describe pod <pod-name>
   
   # Check node capacity
   kubectl describe nodes
   ```

## 📚 Additional Resources

- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)
- [Deployment Strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#deployment-strategies)

## 🎉 Lab Completion

Congratulations! You've completed the Deployments and ReplicaSets lab. You now understand:
- Deployment creation and management
- ReplicaSet relationship with Deployments
- Rolling updates and rollback strategies
- Resource management in Deployments
- Advanced deployment patterns

**Next Lab**: [Lab 03: Services and Networking](../kubernetes/lab-03-services-networking.md)

---

**Happy Learning! ☸️** 