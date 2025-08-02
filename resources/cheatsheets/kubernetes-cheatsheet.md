# Kubernetes Cheat Sheet

## ☸️ Basic Commands

### Cluster Information
```bash
# Get cluster info
kubectl cluster-info

# Get nodes
kubectl get nodes

# Get namespaces
kubectl get namespaces

# Get all resources
kubectl get all

# Get all resources in all namespaces
kubectl get all --all-namespaces
```

### Pod Management
```bash
# List pods
kubectl get pods

# List pods with more details
kubectl get pods -o wide

# Get pod details
kubectl describe pod <pod-name>

# Get pod logs
kubectl logs <pod-name>

# Follow pod logs
kubectl logs -f <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash

# Copy files from/to pod
kubectl cp <pod-name>:/path /local/path
kubectl cp /local/path <pod-name>:/path

# Delete pod
kubectl delete pod <pod-name>

# Force delete pod
kubectl delete pod <pod-name> --grace-period=0 --force
```

### Deployment Management
```bash
# List deployments
kubectl get deployments

# Get deployment details
kubectl describe deployment <deployment-name>

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=3

# Update deployment image
kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Rollback deployment
kubectl rollout undo deployment/<deployment-name>

# Check rollout status
kubectl rollout status deployment/<deployment-name>

# Pause rollout
kubectl rollout pause deployment/<deployment-name>

# Resume rollout
kubectl rollout resume deployment/<deployment-name>
```

### Service Management
```bash
# List services
kubectl get services

# Get service details
kubectl describe service <service-name>

# Port forward to service
kubectl port-forward svc/<service-name> 8080:80

# Delete service
kubectl delete service <service-name>
```

## 📋 Resource Types

### Common Resources
```bash
# Pods
kubectl get pods
kubectl create pod <pod-name>

# Deployments
kubectl get deployments
kubectl create deployment <name> --image=<image>

# Services
kubectl get services
kubectl expose deployment <name> --port=80

# ConfigMaps
kubectl get configmaps
kubectl create configmap <name> --from-file=<file>

# Secrets
kubectl get secrets
kubectl create secret generic <name> --from-literal=<key>=<value>

# PersistentVolumes
kubectl get pv
kubectl get pvc

# Namespaces
kubectl get namespaces
kubectl create namespace <name>
```

### Advanced Resources
```bash
# StatefulSets
kubectl get statefulsets
kubectl describe statefulset <name>

# DaemonSets
kubectl get daemonsets
kubectl describe daemonset <name>

# Jobs
kubectl get jobs
kubectl describe job <name>

# CronJobs
kubectl get cronjobs
kubectl describe cronjob <name>

# Ingress
kubectl get ingress
kubectl describe ingress <name>

# NetworkPolicies
kubectl get networkpolicies
kubectl describe networkpolicy <name>
```

## 🔧 YAML Templates

### Pod Template
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:alpine
    ports:
    - containerPort: 80
    env:
    - name: ENV_VAR
      value: "value"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Deployment Template
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:alpine
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Service Template
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: my-app
```

### ConfigMap Template
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  config.json: |
    {
      "database": {
        "host": "localhost",
        "port": 5432
      }
    }
  app.properties: |
    server.port=8080
    logging.level=INFO
```

### Secret Template
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

## 🌐 Networking

### Service Types
```yaml
# ClusterIP (default)
spec:
  type: ClusterIP

# NodePort
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080

# LoadBalancer
spec:
  type: LoadBalancer

# ExternalName
spec:
  type: ExternalName
  externalName: api.example.com
```

### Ingress Template
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
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

## 💾 Storage

### PersistentVolume Template
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```

### PersistentVolumeClaim Template
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## 🔒 Security

### RBAC Template
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### SecurityContext Template
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: my-container
    image: nginx:alpine
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

## 📊 Monitoring and Debugging

### Resource Monitoring
```bash
# Get resource usage
kubectl top nodes
kubectl top pods

# Get events
kubectl get events --sort-by='.lastTimestamp'

# Get logs from multiple pods
kubectl logs -l app=my-app

# Get logs from previous container
kubectl logs <pod-name> --previous
```

### Debugging Commands
```bash
# Get detailed resource info
kubectl describe <resource> <name>

# Get resource in YAML format
kubectl get <resource> <name> -o yaml

# Get resource in JSON format
kubectl get <resource> <name> -o json

# Edit resource
kubectl edit <resource> <name>

# Apply changes from file
kubectl apply -f <file>

# Delete resource
kubectl delete -f <file>
```

## 🔄 Scaling and Updates

### Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Rolling Update
```bash
# Update deployment
kubectl set image deployment/my-deployment container=new-image:tag

# Check rollout status
kubectl rollout status deployment/my-deployment

# Rollback to previous version
kubectl rollout undo deployment/my-deployment

# Rollback to specific version
kubectl rollout undo deployment/my-deployment --to-revision=2
```

## 🚀 Advanced Features

### Jobs and CronJobs
```yaml
# Job Template
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: my-job
        image: busybox
        command: ["echo", "Hello from job"]
      restartPolicy: Never
  backoffLimit: 4
```

```yaml
# CronJob Template
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: my-cronjob
            image: busybox
            command: ["echo", "Hello from cronjob"]
          restartPolicy: OnFailure
```

### StatefulSet Template
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  serviceName: my-service
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: my-volume
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: my-volume
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

## 🔧 Troubleshooting

### Common Issues
```bash
# Pod stuck in Pending
kubectl describe pod <pod-name>
kubectl get events --sort-by='.lastTimestamp'

# Pod stuck in CrashLoopBackOff
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>

# Service not accessible
kubectl get endpoints <service-name>
kubectl describe service <service-name>

# Image pull errors
kubectl describe pod <pod-name> | grep -A 10 Events
```

### Useful Commands
```bash
# Get all resources in namespace
kubectl get all -n <namespace>

# Get resource with labels
kubectl get pods -l app=my-app

# Get resource with custom output
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# Watch resources
kubectl get pods -w

# Get resource usage
kubectl top pods --containers
```

## 📝 Tips and Tricks

### Useful Aliases
```bash
# Add to ~/.bashrc or ~/.zshrc
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kdp='kubectl delete pod'
alias kds='kubectl delete service'
alias kdd='kubectl delete deployment'
```

### Context Management
```bash
# List contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Get current context
kubectl config current-context

# Set namespace for current context
kubectl config set-context --current --namespace=<namespace>
```

---

**Happy Kubernetes Learning! ☸️** 