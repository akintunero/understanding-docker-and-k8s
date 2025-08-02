# Simple Kubernetes Deployment Example

This example demonstrates how to deploy a web application to Kubernetes with best practices for production environments.

## 🚀 Quick Start

### Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured
- Docker image built and available

### Deploy the Application

1. **Build and push the Docker image:**
   ```bash
   # Build the image
   docker build -t simple-web-app:latest ../docker/simple-web-app/
   
   # If using Minikube, load the image
   minikube image load simple-web-app:latest
   
   # Or push to a registry
   docker tag simple-web-app:latest your-registry/simple-web-app:latest
   docker push your-registry/simple-web-app:latest
   ```

2. **Deploy to Kubernetes:**
   ```bash
   # Apply the deployment
   kubectl apply -f deployment.yaml
   
   # Apply the service
   kubectl apply -f service.yaml
   
   # Apply the ingress (optional)
   kubectl apply -f ingress.yaml
   ```

3. **Verify the deployment:**
   ```bash
   # Check deployment status
   kubectl get deployments
   
   # Check pods
   kubectl get pods -l app=simple-web-app
   
   # Check services
   kubectl get services
   ```

## 📁 File Structure

```
simple-deployment/
├── deployment.yaml    # Main deployment configuration
├── service.yaml       # Service definitions
├── ingress.yaml       # Ingress configuration
└── README.md         # This file
```

## 🔧 Configuration Details

### Deployment Features

#### 1. **High Availability**
- 3 replicas for redundancy
- Rolling update strategy
- Health checks (liveness and readiness probes)

#### 2. **Resource Management**
- CPU and memory requests/limits
- Prevents resource starvation
- Enables proper scheduling

#### 3. **Security**
- Non-root user execution
- Read-only root filesystem
- Dropped capabilities
- Security contexts

#### 4. **Monitoring**
- Health check endpoints
- Structured logging
- Metrics collection ready

### Service Types

#### ClusterIP Service
- Internal cluster communication
- Load balancing across pods
- Default service type

#### NodePort Service
- External access via node ports
- Accessible from outside cluster
- Port 30080 exposed

### Ingress Configuration

#### HTTP Ingress
- Basic HTTP routing
- Host-based routing
- Path-based routing

#### HTTPS Ingress
- TLS termination
- SSL certificate management
- Secure communication

## 🧪 Testing

### Check Application Health
```bash
# Port forward to access the service
kubectl port-forward svc/simple-web-app-service 8080:80

# Test the application
curl http://localhost:8080/health
curl http://localhost:8080/
```

### Load Testing
```bash
# Install hey for load testing
go install github.com/rakyll/hey@latest

# Run load test
hey -n 1000 -c 10 http://localhost:8080/
```

### Monitoring
```bash
# Check pod logs
kubectl logs -l app=simple-web-app

# Check pod metrics
kubectl top pods -l app=simple-web-app

# Check service endpoints
kubectl get endpoints simple-web-app-service
```

## 🔄 Scaling

### Manual Scaling
```bash
# Scale to 5 replicas
kubectl scale deployment simple-web-app --replicas=5

# Check scaling status
kubectl get pods -l app=simple-web-app
```

### Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: simple-web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: simple-web-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## 🚨 Troubleshooting

### Common Issues

1. **Pods not starting**
   ```bash
   # Check pod events
   kubectl describe pod <pod-name>
   
   # Check pod logs
   kubectl logs <pod-name>
   ```

2. **Service not accessible**
   ```bash
   # Check service endpoints
   kubectl get endpoints simple-web-app-service
   
   # Check service configuration
   kubectl describe service simple-web-app-service
   ```

3. **Image pull errors**
   ```bash
   # Check image pull policy
   kubectl describe pod <pod-name>
   
   # Ensure image is available
   docker images | grep simple-web-app
   ```

### Debug Commands
```bash
# Get detailed pod information
kubectl describe pod <pod-name>

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/sh

# Check pod resource usage
kubectl top pod <pod-name>

# View pod events
kubectl get events --sort-by='.lastTimestamp'
```

## 📊 Monitoring and Observability

### Prometheus Metrics
The application exposes metrics at `/metrics` endpoint for Prometheus scraping.

### Grafana Dashboard
Create a dashboard to monitor:
- Pod resource usage
- Request latency
- Error rates
- Throughput

### Alerting
Set up alerts for:
- High CPU/memory usage
- Pod restarts
- Service unavailability
- Error rate spikes

## 🔐 Security Considerations

### Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: simple-web-app-network-policy
spec:
  podSelector:
    matchLabels:
      app: simple-web-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to: []
    ports:
    - protocol: TCP
      port: 53
```

### RBAC
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: simple-web-app-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
```

## 📚 Learning Objectives

After completing this example, you should understand:

- ✅ How to create Kubernetes deployments
- ✅ Service types and their use cases
- ✅ Ingress configuration and routing
- ✅ Resource management and limits
- ✅ Health checks and probes
- ✅ Security best practices
- ✅ Scaling strategies
- ✅ Troubleshooting techniques

## 🔗 Related Examples

- [Docker Application](../docker/simple-web-app/)
- [Multi-Service Application](../../multi-service/)
- [Advanced Kubernetes Features](../advanced-features/)

---

**Happy Kubernetes Learning! ☸️** 