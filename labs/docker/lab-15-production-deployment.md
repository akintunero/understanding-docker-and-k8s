# Lab 15: Production Deployment

## 🎯 Learning Objectives
- Implement production-ready Docker deployments
- Set up CI/CD pipelines with Docker
- Configure monitoring and alerting
- Learn production best practices

## ⏱️ Estimated Time: 90 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of Docker concepts

## 🚀 Lab Exercises

### Exercise 1: Production-Ready Application

1. **Create production application:**
   ```bash
   cat > production-app.py << EOF
   import os
   import time
   import logging
   from flask import Flask, jsonify
   from datetime import datetime
   
   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)
   
   app = Flask(__name__)
   
   @app.route('/')
   def home():
       return jsonify({
           'service': 'production-app',
           'version': os.getenv('APP_VERSION', '1.0.0'),
           'environment': os.getenv('ENVIRONMENT', 'development'),
           'timestamp': datetime.now().isoformat()
       })
   
   @app.route('/health')
   def health():
       return jsonify({'status': 'healthy', 'timestamp': datetime.now().isoformat()})
   
   @app.route('/metrics')
   def metrics():
       return jsonify({
           'uptime': time.time() - start_time,
           'requests_total': request_count
       })
   
   if __name__ == '__main__':
       start_time = time.time()
       request_count = 0
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

2. **Create production Dockerfile:**
   ```bash
   cat > Dockerfile.production << EOF
   FROM python:3.9-slim
   
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY production-app.py .
   
   USER nobody
   EXPOSE 5000
   
   HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \\
     CMD curl -f http://localhost:5000/health || exit 1
   
   CMD ["python", "production-app.py"]
   EOF
   
   cat > requirements.txt << EOF
   flask==2.0.1
   gunicorn==20.1.0
   EOF
   ```

### Exercise 2: Production Docker Compose

1. **Create production stack:**
   ```bash
   cat > docker-compose.production.yml << EOF
   version: '3.8'
   
   services:
     app:
       build:
         context: .
         dockerfile: Dockerfile.production
       ports:
         - "5000:5000"
       environment:
         - ENVIRONMENT=production
         - APP_VERSION=1.0.0
       deploy:
         replicas: 3
         resources:
           limits:
             cpus: '0.5'
             memory: 512M
         restart_policy:
           condition: on-failure
           delay: 5s
           max_attempts: 3
       restart: unless-stopped
   
     nginx:
       image: nginx:alpine
       ports:
         - "80:80"
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf
       depends_on:
         - app
       restart: unless-stopped
   
     prometheus:
       image: prom/prometheus:latest
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus.yml:/etc/prometheus/prometheus.yml
       restart: unless-stopped
   
     grafana:
       image: grafana/grafana:latest
       ports:
         - "3000:3000"
       environment:
         - GF_SECURITY_ADMIN_PASSWORD=admin
       restart: unless-stopped
   EOF
   ```

2. **Create nginx configuration:**
   ```bash
   cat > nginx.conf << EOF
   events {
       worker_connections 1024;
   }
   
   http {
       upstream app_backend {
           server app:5000;
       }
   
       server {
           listen 80;
           
           location / {
               proxy_pass http://app_backend;
               proxy_set_header Host \$host;
               proxy_set_header X-Real-IP \$remote_addr;
           }
       }
   }
   EOF
   ```

### Exercise 3: CI/CD Pipeline

1. **Create GitHub Actions workflow:**
   ```bash
   mkdir -p .github/workflows
   
   cat > .github/workflows/docker-ci-cd.yml << EOF
   name: Docker CI/CD
   
   on:
     push:
       branches: [ main ]
   
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
       - uses: actions/checkout@v2
       - name: Run tests
         run: docker build -f Dockerfile.production -t production-app .
   
     deploy:
       needs: test
       runs-on: ubuntu-latest
       steps:
       - uses: actions/checkout@v2
       - name: Deploy
         run: echo "Deploy to production"
   EOF
   ```

### Exercise 4: Monitoring Setup

1. **Create Prometheus configuration:**
   ```bash
   cat > prometheus.yml << EOF
   global:
     scrape_interval: 15s
   
   scrape_configs:
     - job_name: 'production-app'
       static_configs:
         - targets: ['app:5000']
       metrics_path: '/metrics'
   EOF
   ```

2. **Start production stack:**
   ```bash
   docker-compose -f docker-compose.production.yml up -d
   ```

3. **Test the deployment:**
   ```bash
   curl http://localhost/
   curl http://localhost/health
   curl http://localhost/metrics
   ```

### Exercise 5: Production Best Practices

1. **Security scanning:**
   ```bash
   # Install Trivy
   brew install trivy
   
   # Scan for vulnerabilities
   trivy image production-app
   ```

2. **Resource monitoring:**
   ```bash
   # Monitor container resources
   docker stats
   
   # Check logs
   docker-compose -f docker-compose.production.yml logs -f
   ```

3. **Backup strategy:**
   ```bash
   cat > backup.sh << EOF
   #!/bin/bash
   echo "Creating backup..."
   docker-compose -f docker-compose.production.yml exec app \\
     tar -czf /tmp/backup_\$(date +%Y%m%d_%H%M%S).tar.gz /app
   EOF
   
   chmod +x backup.sh
   ```

## 🧹 Cleanup

1. **Stop production services:**
   ```bash
   docker-compose -f docker-compose.production.yml down -v
   ```

2. **Remove files:**
   ```bash
   rm -f production-app.py Dockerfile.production requirements.txt \
        docker-compose.production.yml nginx.conf prometheus.yml backup.sh
   rm -rf .github/
   ```

## 📚 Key Takeaways

- **Health Checks**: Implement proper health checks
- **Resource Limits**: Set appropriate resource constraints
- **Monitoring**: Use Prometheus and Grafana
- **Security**: Regular vulnerability scanning
- **CI/CD**: Automate deployment pipeline
- **Backup**: Implement backup strategies

## 🎯 Next Steps

- Learn Kubernetes deployment
- Implement service mesh
- Explore cloud-native patterns
- Practice with real production scenarios 