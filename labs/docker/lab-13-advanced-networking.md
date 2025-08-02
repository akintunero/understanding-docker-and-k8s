# Lab 13: Advanced Networking

## 🎯 Learning Objectives
- Create and manage custom Docker networks
- Implement network policies and security
- Understand service mesh concepts
- Configure advanced networking features
- Learn network troubleshooting techniques

## ⏱️ Estimated Time: 80 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of Docker networking
- Knowledge of TCP/IP and network protocols

## 🚀 Lab Exercises

### Exercise 1: Custom Network Types

1. **Create different network types:**
   ```bash
   # Bridge network (default)
   docker network create --driver bridge bridge-network
   
   # Host network
   docker network create --driver host host-network
   
   # Overlay network (for swarm)
   docker network create --driver overlay overlay-network
   
   # Macvlan network
   docker network create --driver macvlan \
     --subnet=192.168.1.0/24 \
     --gateway=192.168.1.1 \
     -o parent=eth0 macvlan-network
   
   # IPvlan network
   docker network create --driver ipvlan \
     --subnet=192.168.2.0/24 \
     --gateway=192.168.2.1 \
     -o parent=eth0 ipvlan-network
   ```

2. **List and inspect networks:**
   ```bash
   # List all networks
   docker network ls
   
   # Inspect bridge network
   docker network inspect bridge-network
   
   # Inspect macvlan network
   docker network inspect macvlan-network
   ```

### Exercise 2: Multi-Container Networking

1. **Create a web application stack:**
   ```bash
   # Create web application
   cat > web-app.py << EOF
   from flask import Flask, jsonify
   import requests
   import os
   
   app = Flask(__name__)
   
   @app.route('/')
   def home():
       return jsonify({
           'service': 'web',
           'message': 'Hello from Web Service!',
           'database_url': os.getenv('DATABASE_URL', 'not set')
       })
   
   @app.route('/api/data')
   def get_data():
       try:
           response = requests.get('http://api:5001/data')
           return response.json()
       except Exception as e:
           return jsonify({'error': str(e)}), 500
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   EOF
   
   # Create API service
   cat > api-service.py << EOF
   from flask import Flask, jsonify
   import psycopg2
   import os
   
   app = Flask(__name__)
   
   @app.route('/data')
   def get_data():
       try:
           conn = psycopg2.connect(os.getenv('DATABASE_URL'))
           cursor = conn.cursor()
           cursor.execute('SELECT version()')
           version = cursor.fetchone()
           cursor.close()
           conn.close()
           
           return jsonify({
               'service': 'api',
               'database_version': version[0],
               'status': 'connected'
           })
       except Exception as e:
           return jsonify({'error': str(e)}), 500
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5001)
   EOF
   ```

2. **Create Dockerfiles:**
   ```bash
   cat > Dockerfile.web << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY web-app.py .
   EXPOSE 5000
   CMD ["python", "web-app.py"]
   EOF
   
   cat > Dockerfile.api << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY api-service.py .
   EXPOSE 5001
   CMD ["python", "api-service.py"]
   EOF
   
   cat > requirements.txt << EOF
   flask==2.0.1
   requests==2.26.0
   psycopg2-binary==2.9.1
   EOF
   ```

3. **Build images:**
   ```bash
   docker build -f Dockerfile.web -t web-app .
   docker build -f Dockerfile.api -t api-service .
   ```

4. **Create custom network and run services:**
   ```bash
   # Create custom network
   docker network create app-network
   
   # Run database
   docker run -d \
     --name postgres \
     --network app-network \
     -e POSTGRES_DB=myapp \
     -e POSTGRES_USER=appuser \
     -e POSTGRES_PASSWORD=secret123 \
     postgres:13-alpine
   
   # Run API service
   docker run -d \
     --name api \
     --network app-network \
     -e DATABASE_URL=postgresql://appuser:secret123@postgres:5432/myapp \
     api-service
   
   # Run web service
   docker run -d \
     --name web \
     --network app-network \
     -e DATABASE_URL=postgresql://appuser:secret123@postgres:5432/myapp \
     -p 5000:5000 \
     web-app
   ```

5. **Test the network connectivity:**
   ```bash
   # Test web service
   curl http://localhost:5000/
   
   # Test API service through web
   curl http://localhost:5000/api/data
   
   # Test direct API access
   docker exec web curl http://api:5001/data
   ```

### Exercise 3: Network Security and Policies

1. **Create network with security policies:**
   ```bash
   # Create isolated network
   docker network create --driver bridge \
     --internal isolated-network
   
   # Create network with custom subnet
   docker network create --driver bridge \
     --subnet=172.20.0.0/16 \
     --gateway=172.20.0.1 \
     secure-network
   ```

2. **Run services with network policies:**
   ```bash
   # Run internal service (no external access)
   docker run -d \
     --name internal-service \
     --network isolated-network \
     nginx:alpine
   
   # Run secure service
   docker run -d \
     --name secure-service \
     --network secure-network \
     --ip=172.20.0.10 \
     nginx:alpine
   
   # Run service with multiple networks
   docker run -d \
     --name multi-network-service \
     --network secure-network \
     --network app-network \
     nginx:alpine
   ```

3. **Test network isolation:**
   ```bash
   # Test internal service (should not have external access)
   docker exec internal-service ping -c 3 google.com
   
   # Test secure service
   docker exec secure-service ping -c 3 172.20.0.1
   
   # Test multi-network service
   docker exec multi-network-service ping -c 3 postgres
   ```

### Exercise 4: Service Discovery

1. **Create service discovery test:**
   ```bash
   cat > service-discovery.py << EOF
   import socket
   import time
   import os
   
   def discover_services():
       services = ['web', 'api', 'postgres', 'redis']
       
       for service in services:
           try:
               # Try to resolve service name
               ip = socket.gethostbyname(service)
               print(f"✓ {service} -> {ip}")
           except socket.gaierror:
               print(f"✗ {service} -> not found")
   
   def test_connectivity():
       services = [
           ('web', 5000),
           ('api', 5001),
           ('postgres', 5432),
           ('redis', 6379)
       ]
       
       for service, port in services:
           try:
               sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
               sock.settimeout(2)
               result = sock.connect_ex((service, port))
               sock.close()
               
               if result == 0:
                   print(f"✓ {service}:{port} -> accessible")
               else:
                   print(f"✗ {service}:{port} -> not accessible")
           except Exception as e:
               print(f"✗ {service}:{port} -> error: {e}")
   
   if __name__ == '__main__':
       print("=== Service Discovery Test ===")
       discover_services()
       print("\\n=== Connectivity Test ===")
       test_connectivity()
   EOF
   ```

2. **Run service discovery test:**
   ```bash
   docker run --rm \
     --network app-network \
     -v $(pwd)/service-discovery.py:/app/service-discovery.py \
     python:3.9-slim \
     python /app/service-discovery.py
   ```

### Exercise 5: Load Balancing and Proxy

1. **Create load balancer configuration:**
   ```bash
   cat > nginx-lb.conf << EOF
   events {
       worker_connections 1024;
   }
   
   http {
       upstream web_backend {
           server web:5000;
           server web2:5000;
           server web3:5000;
       }
   
       upstream api_backend {
           server api:5001;
           server api2:5001;
       }
   
       server {
           listen 80;
           
           location / {
               proxy_pass http://web_backend;
               proxy_set_header Host \$host;
               proxy_set_header X-Real-IP \$remote_addr;
               proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
           }
           
           location /api/ {
               proxy_pass http://api_backend/;
               proxy_set_header Host \$host;
               proxy_set_header X-Real-IP \$remote_addr;
               proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
           }
       }
   }
   EOF
   ```

2. **Create additional web instances:**
   ```bash
   # Run additional web instances
   docker run -d \
     --name web2 \
     --network app-network \
     -e DATABASE_URL=postgresql://appuser:secret123@postgres:5432/myapp \
     web-app
   
   docker run -d \
     --name web3 \
     --network app-network \
     -e DATABASE_URL=postgresql://appuser:secret123@postgres:5432/myapp \
     web-app
   
   # Run additional API instances
   docker run -d \
     --name api2 \
     --network app-network \
     -e DATABASE_URL=postgresql://appuser:secret123@postgres:5432/myapp \
     api-service
   ```

3. **Run load balancer:**
   ```bash
   docker run -d \
     --name load-balancer \
     --network app-network \
     -p 80:80 \
     -v $(pwd)/nginx-lb.conf:/etc/nginx/nginx.conf \
     nginx:alpine
   ```

4. **Test load balancing:**
   ```bash
   # Test multiple requests
   for i in {1..10}; do
     curl -s http://localhost/ | jq -r '.service'
   done
   
   # Test API load balancing
   for i in {1..5}; do
     curl -s http://localhost/api/data | jq -r '.service'
   done
   ```

### Exercise 6: Network Troubleshooting

1. **Create network troubleshooting tools:**
   ```bash
   cat > network-tools.sh << EOF
   #!/bin/bash
   
   echo "=== Network Diagnostics ==="
   
   # Check network interfaces
   echo "Network Interfaces:"
   ip addr show
   
   # Check routing table
   echo "\\nRouting Table:"
   ip route show
   
   # Check DNS resolution
   echo "\\nDNS Resolution:"
   nslookup web
   nslookup api
   nslookup postgres
   
   # Check connectivity
   echo "\\nConnectivity Tests:"
   ping -c 3 web
   ping -c 3 api
   ping -c 3 postgres
   
   # Check ports
   echo "\\nPort Scans:"
   nmap -p 5000,5001,5432,6379 web
   nmap -p 5000,5001,5432,6379 api
   EOF
   
   chmod +x network-tools.sh
   ```

2. **Run network diagnostics:**
   ```bash
   docker run --rm \
     --network app-network \
     -v $(pwd)/network-tools.sh:/app/network-tools.sh \
     --privileged \
     nicolaka/netshoot \
     /app/network-tools.sh
   ```

### Exercise 7: Service Mesh Concepts

1. **Create simple service mesh simulation:**
   ```bash
   cat > service-mesh.py << EOF
   import time
   import random
   import requests
   from flask import Flask, jsonify, request
   
   app = Flask(__name__)
   
   # Circuit breaker simulation
   class CircuitBreaker:
       def __init__(self, failure_threshold=3, timeout=60):
           self.failure_threshold = failure_threshold
           self.timeout = timeout
           self.failure_count = 0
           self.last_failure_time = None
           self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
   
       def call(self, func, *args, **kwargs):
           if self.state == 'OPEN':
               if time.time() - self.last_failure_time > self.timeout:
                   self.state = 'HALF_OPEN'
               else:
                   raise Exception('Circuit breaker is OPEN')
           
           try:
               result = func(*args, **kwargs)
               if self.state == 'HALF_OPEN':
                   self.state = 'CLOSED'
                   self.failure_count = 0
               return result
           except Exception as e:
               self.failure_count += 1
               self.last_failure_time = time.time()
               
               if self.failure_count >= self.failure_threshold:
                   self.state = 'OPEN'
               
               raise e
   
   # Create circuit breaker
   cb = CircuitBreaker()
   
   @app.route('/api/proxy')
   def proxy_request():
       try:
           # Simulate service mesh proxy behavior
           target = request.args.get('target', 'api')
           endpoint = request.args.get('endpoint', '/data')
           
           # Add service mesh headers
           headers = {
               'X-Request-ID': request.headers.get('X-Request-ID', ''),
               'X-Trace-ID': request.headers.get('X-Trace-ID', ''),
               'X-Span-ID': request.headers.get('X-Span-ID', '')
           }
           
           # Circuit breaker call
           response = cb.call(requests.get, f'http://{target}:5001{endpoint}', headers=headers)
           
           return jsonify({
               'proxy': True,
               'target': target,
               'response': response.json(),
               'circuit_breaker_state': cb.state
           })
       except Exception as e:
           return jsonify({'error': str(e), 'circuit_breaker_state': cb.state}), 500
   
   @app.route('/health')
   def health():
       return jsonify({'status': 'healthy', 'circuit_breaker_state': cb.state})
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5002)
   EOF
   ```

2. **Create service mesh Dockerfile:**
   ```bash
   cat > Dockerfile.mesh << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY service-mesh.py .
   EXPOSE 5002
   CMD ["python", "service-mesh.py"]
   EOF
   ```

3. **Build and run service mesh:**
   ```bash
   docker build -f Dockerfile.mesh -t service-mesh .
   
   docker run -d \
     --name mesh-proxy \
     --network app-network \
     -p 5002:5002 \
     service-mesh
   ```

4. **Test service mesh functionality:**
   ```bash
   # Test normal request
   curl "http://localhost:5002/api/proxy?target=api&endpoint=/data"
   
   # Test with tracing headers
   curl -H "X-Request-ID: test-123" \
        -H "X-Trace-ID: trace-456" \
        -H "X-Span-ID: span-789" \
        "http://localhost:5002/api/proxy?target=api&endpoint=/data"
   
   # Check health
   curl http://localhost:5002/health
   ```

## 🧹 Cleanup

1. **Stop and remove containers:**
   ```bash
   docker stop web web2 web3 api api2 postgres load-balancer mesh-proxy
   docker rm web web2 web3 api api2 postgres load-balancer mesh-proxy
   ```

2. **Remove networks:**
   ```bash
   docker network rm bridge-network host-network overlay-network \
     macvlan-network ipvlan-network app-network isolated-network \
     secure-network
   ```

3. **Remove images:**
   ```bash
   docker rmi web-app api-service service-mesh
   ```

4. **Clean up files:**
   ```bash
   rm -f web-app.py api-service.py service-discovery.py service-mesh.py \
        Dockerfile.web Dockerfile.api Dockerfile.mesh requirements.txt \
        nginx-lb.conf network-tools.sh
   ```

## 📚 Key Takeaways

- **Custom Networks**: Use appropriate network types for different use cases
- **Service Discovery**: Docker provides automatic DNS resolution
- **Network Security**: Implement proper network isolation and policies
- **Load Balancing**: Use reverse proxies for traffic distribution
- **Service Mesh**: Understand circuit breakers and tracing concepts
- **Troubleshooting**: Use network diagnostic tools effectively

## 🔍 Troubleshooting

### Common Issues:
1. **Service not reachable**: Check network connectivity and DNS resolution
2. **Port conflicts**: Verify port mappings and service configurations
3. **Network isolation**: Ensure proper network policies and access controls
4. **Load balancing issues**: Check proxy configuration and backend health

### Debug Commands:
```bash
# Check network connectivity
docker exec <container> ping <service>

# Inspect network configuration
docker network inspect <network>

# Check DNS resolution
docker exec <container> nslookup <service>

# Monitor network traffic
docker exec <container> tcpdump -i any
```

## 🎯 Next Steps

- Learn about Kubernetes networking concepts
- Explore Istio or Linkerd service mesh
- Implement network policies and security
- Study container network interfaces (CNI)
- Practice with production networking scenarios 