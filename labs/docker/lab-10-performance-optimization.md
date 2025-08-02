# Lab 10: Performance Optimization

## 🎯 Learning Objectives
- Optimize Docker images for size and performance
- Set and monitor resource limits
- Implement performance monitoring
- Use multi-stage builds for optimization
- Understand Docker performance best practices

## ⏱️ Estimated Time: 55 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of Docker images and containers
- Knowledge of resource monitoring concepts

## 🚀 Lab Exercises

### Exercise 1: Image Size Optimization

1. **Create a basic Dockerfile:**
   ```bash
   cat > Dockerfile.basic << EOF
   FROM ubuntu:20.04
   
   RUN apt-get update && apt-get install -y python3 python3-pip
   
   COPY requirements.txt .
   RUN pip3 install -r requirements.txt
   
   COPY app.py .
   
   CMD ["python3", "app.py"]
   EOF
   ```

2. **Create requirements.txt:**
   ```bash
   cat > requirements.txt << EOF
   flask==2.0.1
   requests==2.26.0
   EOF
   ```

3. **Create a simple Flask app:**
   ```bash
   cat > app.py << EOF
   from flask import Flask
   import time
   
   app = Flask(__name__)
   
   @app.route('/')
   def hello():
       return 'Hello, Optimized World!'
   
   @app.route('/health')
   def health():
       return {'status': 'healthy', 'timestamp': time.time()}
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=8080)
   EOF
   ```

4. **Build and check image size:**
   ```bash
   docker build -f Dockerfile.basic -t basic-app .
   docker images basic-app
   ```

5. **Create an optimized Dockerfile:**
   ```bash
   cat > Dockerfile.optimized << EOF
   FROM python:3.9-slim
   
   WORKDIR /app
   
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   
   COPY app.py .
   
   USER nobody
   
   EXPOSE 8080
   
   CMD ["python3", "app.py"]
   EOF
   ```

6. **Build optimized image and compare:**
   ```bash
   docker build -f Dockerfile.optimized -t optimized-app .
   docker images | grep app
   ```

### Exercise 2: Multi-Stage Builds

1. **Create a multi-stage Dockerfile:**
   ```bash
   cat > Dockerfile.multistage << EOF
   # Build stage
   FROM node:16-alpine AS builder
   
   WORKDIR /app
   
   COPY package*.json ./
   RUN npm ci --only=production
   
   COPY . .
   RUN npm run build
   
   # Production stage
   FROM nginx:alpine
   
   COPY --from=builder /app/dist /usr/share/nginx/html
   COPY nginx.conf /etc/nginx/nginx.conf
   
   EXPOSE 80
   
   CMD ["nginx", "-g", "daemon off;"]
   EOF
   ```

2. **Create package.json:**
   ```bash
   cat > package.json << EOF
   {
     "name": "optimized-app",
     "version": "1.0.0",
     "scripts": {
       "build": "echo 'Build completed' && mkdir -p dist && echo '<html><body><h1>Optimized App</h1></body></html>' > dist/index.html"
     },
     "dependencies": {
       "express": "^4.17.1"
     }
   }
   EOF
   ```

3. **Create nginx.conf:**
   ```bash
   cat > nginx.conf << EOF
   events {
       worker_connections 1024;
   }
   
   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
       
       gzip on;
       gzip_types text/plain text/css application/json application/javascript;
       
       server {
           listen 80;
           server_name localhost;
           
           location / {
               root   /usr/share/nginx/html;
               index  index.html index.htm;
           }
       }
   }
   EOF
   ```

4. **Build multi-stage image:**
   ```bash
   docker build -f Dockerfile.multistage -t multistage-app .
   docker images multistage-app
   ```

### Exercise 3: Resource Limits and Monitoring

1. **Create a resource-intensive application:**
   ```bash
   cat > resource-app.py << EOF
   import time
   import threading
   import os
   
   def cpu_intensive():
       while True:
           _ = sum(i * i for i in range(1000))
   
   def memory_intensive():
       data = []
       while True:
           data.append('x' * 1024 * 1024)  # 1MB
           time.sleep(0.1)
   
   if __name__ == '__main__':
       print(f"Process ID: {os.getpid()}")
       
       # Start CPU intensive thread
       cpu_thread = threading.Thread(target=cpu_intensive)
       cpu_thread.daemon = True
       cpu_thread.start()
       
       # Start memory intensive thread
       mem_thread = threading.Thread(target=memory_intensive)
       mem_thread.daemon = True
       mem_thread.start()
       
       print("Resource intensive app started...")
       time.sleep(3600)  # Run for 1 hour
   EOF
   ```

2. **Run container with resource limits:**
   ```bash
   docker run -d \
     --name resource-limited \
     --memory=256m \
     --cpus=0.5 \
     --pids-limit=50 \
     python:3.9-slim python3 /app/resource-app.py
   ```

3. **Monitor resource usage:**
   ```bash
   # Real-time monitoring
   docker stats resource-limited
   
   # Check container details
   docker inspect resource-limited | jq '.[0].HostConfig.Memory'
   docker inspect resource-limited | jq '.[0].HostConfig.CpuQuota'
   ```

### Exercise 4: Performance Monitoring

1. **Install monitoring tools:**
   ```bash
   # For macOS
   brew install htop
   
   # For Linux
   sudo apt-get install htop
   ```

2. **Create a monitoring script:**
   ```bash
   cat > monitor.sh << EOF
   #!/bin/bash
   
   echo "=== Docker Performance Monitor ==="
   echo "Time: \$(date)"
   echo
   
   echo "=== Container Stats ==="
   docker stats --no-stream
   echo
   
   echo "=== System Resources ==="
   echo "CPU Usage:"
   top -l 1 | grep "CPU usage" || top -bn1 | grep "Cpu(s)"
   echo
   
   echo "Memory Usage:"
   free -h 2>/dev/null || vm_stat
   echo
   
   echo "Disk Usage:"
   df -h
   echo
   
   echo "=== Docker Info ==="
   docker info | grep -E "(Containers|Images|Storage Driver)"
   EOF
   
   chmod +x monitor.sh
   ```

3. **Run monitoring:**
   ```bash
   ./monitor.sh
   ```

### Exercise 5: Layer Caching Optimization

1. **Create an inefficient Dockerfile:**
   ```bash
   cat > Dockerfile.inefficient << EOF
   FROM ubuntu:20.04
   
   RUN apt-get update
   RUN apt-get install -y python3
   RUN apt-get install -y python3-pip
   
   COPY requirements.txt .
   RUN pip3 install -r requirements.txt
   
   COPY app.py .
   COPY config.py .
   COPY utils.py .
   
   CMD ["python3", "app.py"]
   EOF
   ```

2. **Create additional files:**
   ```bash
   cat > config.py << EOF
   DEBUG = True
   HOST = '0.0.0.0'
   PORT = 8080
   EOF
   
   cat > utils.py << EOF
   def get_timestamp():
       import time
       return time.time()
   EOF
   ```

3. **Create an optimized Dockerfile:**
   ```bash
   cat > Dockerfile.efficient << EOF
   FROM ubuntu:20.04
   
   # Combine package installation
   RUN apt-get update && apt-get install -y python3 python3-pip
   
   # Copy requirements first for better caching
   COPY requirements.txt .
   RUN pip3 install -r requirements.txt
   
   # Copy application files
   COPY app.py config.py utils.py ./
   
   CMD ["python3", "app.py"]
   EOF
   ```

4. **Build and compare build times:**
   ```bash
   time docker build -f Dockerfile.inefficient -t inefficient-app .
   time docker build -f Dockerfile.efficient -t efficient-app .
   ```

### Exercise 6: Docker Compose Performance

1. **Create a performance-focused docker-compose.yml:**
   ```bash
   cat > docker-compose.performance.yml << EOF
   version: '3.8'
   
   services:
     web:
       build:
         context: .
         dockerfile: Dockerfile.optimized
       deploy:
         resources:
           limits:
             cpus: '0.5'
             memory: 256M
           reservations:
             cpus: '0.25'
             memory: 128M
       ports:
         - "8080:8080"
       restart: unless-stopped
   
     db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: myapp
         POSTGRES_USER: appuser
         POSTGRES_PASSWORD: secret123
       deploy:
         resources:
           limits:
             cpus: '1.0'
             memory: 512M
       volumes:
         - postgres_data:/var/lib/postgresql/data
       restart: unless-stopped
   
     redis:
       image: redis:alpine
       command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
       deploy:
         resources:
           limits:
             cpus: '0.25'
             memory: 256M
       restart: unless-stopped
   
     monitor:
       image: prom/prometheus:latest
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus.yml:/etc/prometheus/prometheus.yml
       deploy:
         resources:
           limits:
             cpus: '0.5'
             memory: 256M
       restart: unless-stopped
   
   volumes:
     postgres_data:
   EOF
   ```

2. **Create Prometheus configuration:**
   ```bash
   cat > prometheus.yml << EOF
   global:
     scrape_interval: 15s
   
   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']
   EOF
   ```

3. **Start the performance-optimized stack:**
   ```bash
   docker-compose -f docker-compose.performance.yml up -d
   ```

4. **Monitor the stack:**
   ```bash
   docker-compose -f docker-compose.performance.yml ps
   docker-compose -f docker-compose.performance.yml logs -f
   ```

### Exercise 7: Performance Testing

1. **Create a load testing script:**
   ```bash
   cat > load-test.sh << EOF
   #!/bin/bash
   
   echo "Starting load test..."
   
   # Test web service
   for i in {1..100}; do
     curl -s http://localhost:8080/ > /dev/null
     echo "Request \$i completed"
   done
   
   echo "Load test completed!"
   EOF
   
   chmod +x load-test.sh
   ```

2. **Run performance tests:**
   ```bash
   # Test response time
   time curl -s http://localhost:8080/
   
   # Run load test
   ./load-test.sh
   
   # Check resource usage during load
   docker stats --no-stream
   ```

## 🧹 Cleanup

1. **Stop and remove containers:**
   ```bash
   docker stop resource-limited
   docker rm resource-limited
   ```

2. **Stop Docker Compose services:**
   ```bash
   docker-compose -f docker-compose.performance.yml down -v
   ```

3. **Remove images:**
   ```bash
   docker rmi basic-app optimized-app multistage-app inefficient-app efficient-app
   ```

4. **Clean up files:**
   ```bash
   rm -f Dockerfile.* app.py requirements.txt package.json nginx.conf \
        resource-app.py monitor.sh config.py utils.py \
        docker-compose.performance.yml prometheus.yml load-test.sh
   ```

## 📚 Key Takeaways

- **Image Optimization**: Use smaller base images and multi-stage builds
- **Layer Caching**: Order Dockerfile instructions for optimal caching
- **Resource Limits**: Set appropriate CPU and memory limits
- **Monitoring**: Implement performance monitoring and alerting
- **Multi-Stage Builds**: Reduce final image size significantly
- **Best Practices**: Combine package installations and optimize layer order

## 🔍 Troubleshooting

### Common Issues:
1. **Large image sizes**: Use multi-stage builds and smaller base images
2. **Slow builds**: Optimize layer caching and reduce dependencies
3. **High resource usage**: Set appropriate resource limits
4. **Performance degradation**: Monitor and optimize resource allocation

### Debug Commands:
```bash
# Check image layers
docker history <image>

# Monitor resource usage
docker stats <container>

# Analyze image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Check build cache
docker system df
```

## 🎯 Next Steps

- Implement performance monitoring in production
- Learn about Docker BuildKit for faster builds
- Explore container orchestration performance optimization
- Study Kubernetes resource management
- Practice with production-grade monitoring tools 