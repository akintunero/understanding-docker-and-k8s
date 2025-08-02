# Project 01: Hello World Web App

A simple web application demonstrating basic Docker and Kubernetes concepts.

## 🚀 Quick Start

### Docker Setup
```bash
# Create application
mkdir hello-world-app && cd hello-world-app

# Create HTML file
cat << EOF > index.html
<!DOCTYPE html>
<html>
<head><title>Hello World</title></head>
<body>
    <h1>Hello from Docker!</h1>
    <p>Container: <span id="hostname"></span></p>
    <script>document.getElementById('hostname').textContent = window.location.hostname;</script>
</body>
</html>
EOF

# Create Dockerfile
cat << EOF > Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Build and run
docker build -t hello-world-app .
docker run -d -p 8080:80 --name hello-world hello-world-app
```

### Kubernetes Deployment
```bash
# Create deployment
cat << EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world-app
  template:
    metadata:
      labels:
        app: hello-world-app
    spec:
      containers:
      - name: app
        image: hello-world-app:latest
        ports:
        - containerPort: 80
EOF

# Create service
cat << EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
spec:
  selector:
    app: hello-world-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF

# Deploy
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## 🎯 Learning Objectives
- Basic containerization with Docker
- Simple Kubernetes deployment
- Web application containerization
- Service configuration

**Access**: http://localhost:8080 (Docker) or via kubectl port-forward

---

**Happy Learning! 🐳☸️** 