# Lab 14: Monitoring and Debugging

## 🎯 Learning Objectives
- Monitor container performance and resources
- Implement comprehensive logging strategies
- Use debugging tools and techniques
- Set up monitoring and alerting
- Troubleshoot container issues effectively

## ⏱️ Estimated Time: 70 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of Docker containers
- Knowledge of monitoring concepts

## 🚀 Lab Exercises

### Exercise 1: Container Monitoring

1. **Create a resource-intensive application:**
   ```bash
   cat > monitoring-app.py << EOF
   import time
   import threading
   import random
   import os
   from flask import Flask, jsonify
   
   app = Flask(__name__)
   
   # Global variables for resource usage
   cpu_data = []
   memory_data = []
   
   def cpu_intensive_task():
       while True:
           # Simulate CPU intensive work
           _ = sum(i * i for i in range(10000))
           cpu_data.append(time.time())
           time.sleep(0.1)
   
   def memory_intensive_task():
       data = []
       while True:
           # Allocate memory
           data.append('x' * 1024 * 1024)  # 1MB
           memory_data.append(len(data))
           time.sleep(1)
   
   def network_task():
       while True:
           # Simulate network activity
           time.sleep(random.uniform(0.5, 2.0))
   
   @app.route('/')
   def home():
       return jsonify({
           'service': 'monitoring-app',
           'status': 'running',
           'cpu_samples': len(cpu_data),
           'memory_usage': len(memory_data) if memory_data else 0
       })
   
   @app.route('/health')
   def health():
       return jsonify({
           'status': 'healthy',
           'timestamp': time.time(),
           'pid': os.getpid()
       })
   
   @app.route('/metrics')
   def metrics():
       return jsonify({
           'cpu_samples': len(cpu_data),
           'memory_usage_mb': len(memory_data) if memory_data else 0,
           'uptime': time.time() - start_time
       })
   
   if __name__ == '__main__':
       start_time = time.time()
       
       # Start background tasks
       threading.Thread(target=cpu_intensive_task, daemon=True).start()
       threading.Thread(target=memory_intensive_task, daemon=True).start()
       threading.Thread(target=network_task, daemon=True).start()
       
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

2. **Create Dockerfile:**
   ```bash
   cat > Dockerfile.monitoring << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY monitoring-app.py .
   EXPOSE 5000
   CMD ["python", "monitoring-app.py"]
   EOF
   
   cat > requirements.txt << EOF
   flask==2.0.1
   psutil==5.8.0
   EOF
   ```

3. **Build and run the monitoring application:**
   ```bash
   docker build -f Dockerfile.monitoring -t monitoring-app .
   
   docker run -d \
     --name monitoring-demo \
     --memory=512m \
     --cpus=1.0 \
     -p 5000:5000 \
     monitoring-app
   ```

4. **Monitor container resources:**
   ```bash
   # Real-time monitoring
   docker stats monitoring-demo
   
   # Monitor specific metrics
   docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"
   
   # Monitor with custom format
   docker stats --no-stream --format "{{.Container}}: CPU {{.CPUPerc}}, Memory {{.MemUsage}}"
   ```

### Exercise 2: Advanced Monitoring with Prometheus

1. **Create Prometheus configuration:**
   ```bash
   cat > prometheus.yml << EOF
   global:
     scrape_interval: 15s
     evaluation_interval: 15s
   
   rule_files:
     # - "first_rules.yml"
     # - "second_rules.yml"
   
   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']
   
     - job_name: 'docker-app'
       static_configs:
         - targets: ['host.docker.internal:5000']
       metrics_path: '/metrics'
       scrape_interval: 5s
   
     - job_name: 'cadvisor'
       static_configs:
         - targets: ['cadvisor:8080']
   EOF
   ```

2. **Create enhanced monitoring application:**
   ```bash
   cat > enhanced-monitoring.py << EOF
   import time
   import threading
   import random
   import os
   import psutil
   from flask import Flask, jsonify, Response
   
   app = Flask(__name__)
   
   # Metrics storage
   metrics = {
       'cpu_usage': [],
       'memory_usage': [],
       'request_count': 0,
       'error_count': 0
   }
   
   def collect_system_metrics():
       while True:
           try:
               cpu_percent = psutil.cpu_percent(interval=1)
               memory = psutil.virtual_memory()
               
               metrics['cpu_usage'].append({
                   'timestamp': time.time(),
                   'value': cpu_percent
               })
               
               metrics['memory_usage'].append({
                   'timestamp': time.time(),
                   'value': memory.percent
               })
               
               # Keep only last 100 samples
               if len(metrics['cpu_usage']) > 100:
                   metrics['cpu_usage'] = metrics['cpu_usage'][-100:]
               if len(metrics['memory_usage']) > 100:
                   metrics['memory_usage'] = metrics['memory_usage'][-100:]
                   
           except Exception as e:
               print(f"Error collecting metrics: {e}")
           
           time.sleep(5)
   
   @app.route('/')
   def home():
       metrics['request_count'] += 1
       return jsonify({
           'service': 'enhanced-monitoring',
           'status': 'running',
           'request_count': metrics['request_count']
       })
   
   @app.route('/metrics')
   def prometheus_metrics():
       current_time = time.time()
       
       # Generate Prometheus format metrics
       prometheus_metrics = f"""# HELP app_requests_total Total number of requests
# TYPE app_requests_total counter
app_requests_total {metrics['request_count']}
# HELP app_errors_total Total number of errors
# TYPE app_errors_total counter
app_errors_total {metrics['error_count']}
# HELP app_cpu_usage_percent Current CPU usage percentage
# TYPE app_cpu_usage_percent gauge
app_cpu_usage_percent {metrics['cpu_usage'][-1]['value'] if metrics['cpu_usage'] else 0}
# HELP app_memory_usage_percent Current memory usage percentage
# TYPE app_memory_usage_percent gauge
app_memory_usage_percent {metrics['memory_usage'][-1]['value'] if metrics['memory_usage'] else 0}
# HELP app_uptime_seconds Application uptime in seconds
# TYPE app_uptime_seconds gauge
app_uptime_seconds {current_time - start_time}
"""
       
       return Response(prometheus_metrics, mimetype='text/plain')
   
   @app.route('/health')
   def health():
       return jsonify({
           'status': 'healthy',
           'timestamp': time.time(),
           'pid': os.getpid()
       })
   
   @app.route('/error')
   def trigger_error():
       metrics['error_count'] += 1
       return jsonify({'error': 'Simulated error'}), 500
   
   if __name__ == '__main__':
       start_time = time.time()
       
       # Start metrics collection
       threading.Thread(target=collect_system_metrics, daemon=True).start()
       
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

3. **Create Docker Compose for monitoring stack:**
   ```bash
   cat > docker-compose.monitoring.yml << EOF
   version: '3.8'
   
   services:
     app:
       build:
         context: .
         dockerfile: Dockerfile.monitoring
       ports:
         - "5000:5000"
       environment:
         - PYTHONUNBUFFERED=1
       deploy:
         resources:
           limits:
             cpus: '1.0'
             memory: 512M
       restart: unless-stopped
   
     prometheus:
       image: prom/prometheus:latest
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus.yml:/etc/prometheus/prometheus.yml
       command:
         - '--config.file=/etc/prometheus/prometheus.yml'
         - '--storage.tsdb.path=/prometheus'
         - '--web.console.libraries=/etc/prometheus/console_libraries'
         - '--web.console.templates=/etc/prometheus/consoles'
         - '--storage.tsdb.retention.time=200h'
         - '--web.enable-lifecycle'
       restart: unless-stopped
   
     grafana:
       image: grafana/grafana:latest
       ports:
         - "3000:3000"
       environment:
         - GF_SECURITY_ADMIN_PASSWORD=admin
       volumes:
         - grafana-storage:/var/lib/grafana
       restart: unless-stopped
   
     cadvisor:
       image: gcr.io/cadvisor/cadvisor:latest
       ports:
         - "8080:8080"
       volumes:
         - /:/rootfs:ro
         - /var/run:/var/run:ro
         - /sys:/sys:ro
         - /var/lib/docker/:/var/lib/docker:ro
         - /dev/disk/:/dev/disk:ro
       devices:
         - /dev/kmsg:/dev/kmsg
       restart: unless-stopped
   
   volumes:
     grafana-storage:
   EOF
   ```

4. **Start the monitoring stack:**
   ```bash
   docker-compose -f docker-compose.monitoring.yml up -d
   ```

5. **Test the monitoring:**
   ```bash
   # Generate some load
   for i in {1..50}; do
     curl -s http://localhost:5000/ > /dev/null
     sleep 0.1
   done
   
   # Trigger some errors
   for i in {1..10}; do
     curl -s http://localhost:5000/error > /dev/null
     sleep 0.1
   done
   
   # Check metrics
   curl http://localhost:5000/metrics
   ```

### Exercise 3: Logging Strategies

1. **Create a logging application:**
   ```bash
   cat > logging-app.py << EOF
   import logging
   import time
   import random
   import json
   from flask import Flask, jsonify, request
   from datetime import datetime
   
   # Configure logging
   logging.basicConfig(
       level=logging.INFO,
       format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
   )
   logger = logging.getLogger(__name__)
   
   app = Flask(__name__)
   
   # Request counter
   request_count = 0
   
   @app.before_request
   def log_request():
       global request_count
       request_count += 1
       
       logger.info(f"Request {request_count}: {request.method} {request.path} from {request.remote_addr}")
   
   @app.after_request
   def log_response(response):
       logger.info(f"Response: {response.status_code} - {response.content_length} bytes")
       return response
   
   @app.route('/')
   def home():
       logger.info("Home endpoint accessed")
       return jsonify({
           'message': 'Hello from Logging App!',
           'timestamp': datetime.now().isoformat(),
           'request_count': request_count
       })
   
   @app.route('/api/data')
   def get_data():
       logger.info("Data endpoint accessed")
       
       # Simulate some processing
       time.sleep(random.uniform(0.1, 0.5))
       
       data = {
           'id': random.randint(1, 1000),
           'name': f'Item {random.randint(1, 100)}',
           'value': random.uniform(0, 100),
           'timestamp': datetime.now().isoformat()
       }
       
       logger.info(f"Generated data: {json.dumps(data)}")
       return jsonify(data)
   
   @app.route('/api/error')
   def trigger_error():
       logger.error("Error endpoint accessed - simulating error")
       
       # Simulate different error types
       error_type = random.choice(['timeout', 'database', 'network', 'validation'])
       
       if error_type == 'timeout':
           logger.error("Database connection timeout")
           return jsonify({'error': 'Database timeout'}), 500
       elif error_type == 'database':
           logger.error("Database query failed")
           return jsonify({'error': 'Database error'}), 500
       elif error_type == 'network':
           logger.error("Network connection failed")
           return jsonify({'error': 'Network error'}), 503
       else:
           logger.error("Validation error")
           return jsonify({'error': 'Validation error'}), 400
   
   @app.route('/health')
   def health():
       logger.info("Health check performed")
       return jsonify({
           'status': 'healthy',
           'timestamp': datetime.now().isoformat(),
           'request_count': request_count
       })
   
   if __name__ == '__main__':
       logger.info("Starting logging application")
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

2. **Create logging Dockerfile:**
   ```bash
   cat > Dockerfile.logging << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY logging-app.py .
   EXPOSE 5000
   CMD ["python", "logging-app.py"]
   EOF
   ```

3. **Run logging application:**
   ```bash
   docker build -f Dockerfile.logging -t logging-app .
   
   docker run -d \
     --name logging-demo \
     -p 5001:5000 \
     logging-app
   ```

4. **Test logging:**
   ```bash
   # Generate various log entries
   curl http://localhost:5001/
   curl http://localhost:5001/api/data
   curl http://localhost:5001/api/error
   curl http://localhost:5001/health
   
   # View logs
   docker logs logging-demo
   
   # Follow logs in real-time
   docker logs -f logging-demo
   ```

### Exercise 4: Debugging Tools

1. **Create debugging utilities:**
   ```bash
   cat > debug-tools.sh << EOF
   #!/bin/bash
   
   echo "=== Docker Debugging Tools ==="
   
   # Container inspection
   echo "\\n=== Container Inspection ==="
   docker inspect logging-demo | jq '.[0].State'
   
   # Process inspection
   echo "\\n=== Process Inspection ==="
   docker exec logging-demo ps aux
   
   # Network inspection
   echo "\\n=== Network Inspection ==="
   docker exec logging-demo netstat -tulpn
   
   # File system inspection
   echo "\\n=== File System Inspection ==="
   docker exec logging-demo df -h
   docker exec logging-demo ls -la /app
   
   # Environment variables
   echo "\\n=== Environment Variables ==="
   docker exec logging-demo env
   
   # Resource usage
   echo "\\n=== Resource Usage ==="
   docker exec logging-demo cat /proc/1/status | grep -E "(VmRSS|VmSize|Threads)"
   EOF
   
   chmod +x debug-tools.sh
   ```

2. **Run debugging tools:**
   ```bash
   ./debug-tools.sh
   ```

3. **Interactive debugging:**
   ```bash
   # Enter container for debugging
   docker exec -it logging-demo /bin/bash
   
   # Inside container, you can:
   # - Check processes: ps aux
   # - Check network: netstat -tulpn
   # - Check logs: tail -f /var/log/app.log
   # - Check resources: top
   # - Check files: ls -la /app
   ```

### Exercise 5: Performance Profiling

1. **Create performance profiling application:**
   ```bash
   cat > profiling-app.py << EOF
   import time
   import cProfile
   import pstats
   import io
   from flask import Flask, jsonify
   import threading
   import random
   
   app = Flask(__name__)
   
   # Performance metrics
   performance_data = {
       'slow_operations': 0,
       'fast_operations': 0,
       'total_time': 0
   }
   
   def slow_operation():
       """Simulate a slow operation"""
       time.sleep(random.uniform(0.5, 2.0))
       performance_data['slow_operations'] += 1
       return "Slow operation completed"
   
   def fast_operation():
       """Simulate a fast operation"""
       time.sleep(random.uniform(0.01, 0.1))
       performance_data['fast_operations'] += 1
       return "Fast operation completed"
   
   @app.route('/')
   def home():
       return jsonify({
           'message': 'Performance Profiling App',
           'endpoints': {
               '/fast': 'Fast operation',
               '/slow': 'Slow operation',
               '/profile': 'Run profiling',
               '/metrics': 'Performance metrics'
           }
       })
   
   @app.route('/fast')
   def fast():
       start_time = time.time()
       result = fast_operation()
       end_time = time.time()
       
       performance_data['total_time'] += (end_time - start_time)
       
       return jsonify({
           'result': result,
           'duration': end_time - start_time
       })
   
   @app.route('/slow')
   def slow():
       start_time = time.time()
       result = slow_operation()
       end_time = time.time()
       
       performance_data['total_time'] += (end_time - start_time)
       
       return jsonify({
           'result': result,
           'duration': end_time - start_time
       })
   
   @app.route('/profile')
   def profile():
       # Create profiler
       pr = cProfile.Profile()
       pr.enable()
       
       # Run some operations
       for _ in range(10):
           fast_operation()
           slow_operation()
       
       pr.disable()
       
       # Get stats
       s = io.StringIO()
       ps = pstats.Stats(pr, stream=s).sort_stats('cumulative')
       ps.print_stats()
       
       return jsonify({
           'profiling_results': s.getvalue(),
           'operations_completed': 20
       })
   
   @app.route('/metrics')
   def metrics():
       return jsonify({
           'slow_operations': performance_data['slow_operations'],
           'fast_operations': performance_data['fast_operations'],
           'total_time': performance_data['total_time'],
           'average_time': performance_data['total_time'] / max(1, performance_data['slow_operations'] + performance_data['fast_operations'])
       })
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

2. **Create profiling Dockerfile:**
   ```bash
   cat > Dockerfile.profiling << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY profiling-app.py .
   EXPOSE 5000
   CMD ["python", "profiling-app.py"]
   EOF
   ```

3. **Run profiling application:**
   ```bash
   docker build -f Dockerfile.profiling -t profiling-app .
   
   docker run -d \
     --name profiling-demo \
     -p 5002:5000 \
     profiling-app
   ```

4. **Test performance profiling:**
   ```bash
   # Test fast operations
   for i in {1..5}; do
     curl -s http://localhost:5002/fast
     echo
   done
   
   # Test slow operations
   for i in {1..3}; do
     curl -s http://localhost:5002/slow
     echo
   done
   
   # Run profiling
   curl http://localhost:5002/profile
   
   # Check metrics
   curl http://localhost:5002/metrics
   ```

### Exercise 6: Alerting and Monitoring

1. **Create alerting configuration:**
   ```bash
   cat > alerting-rules.yml << EOF
   groups:
   - name: docker-app
     rules:
     - alert: HighCPUUsage
       expr: app_cpu_usage_percent > 80
       for: 2m
       labels:
         severity: warning
       annotations:
         summary: "High CPU usage detected"
         description: "CPU usage is above 80% for 2 minutes"
   
     - alert: HighMemoryUsage
       expr: app_memory_usage_percent > 85
       for: 2m
       labels:
         severity: warning
       annotations:
         summary: "High memory usage detected"
         description: "Memory usage is above 85% for 2 minutes"
   
     - alert: HighErrorRate
       expr: rate(app_errors_total[5m]) > 0.1
       for: 1m
       labels:
         severity: critical
       annotations:
         summary: "High error rate detected"
         description: "Error rate is above 0.1 errors per second"
   
     - alert: ServiceDown
       expr: up{job="docker-app"} == 0
       for: 1m
       labels:
         severity: critical
       annotations:
         summary: "Service is down"
         description: "The application service is not responding"
   EOF
   ```

2. **Update Prometheus configuration with alerting:**
   ```bash
   cat > prometheus-with-alerts.yml << EOF
   global:
     scrape_interval: 15s
     evaluation_interval: 15s
   
   rule_files:
     - "alerting-rules.yml"
   
   alerting:
     alertmanagers:
       - static_configs:
           - targets:
             - alertmanager:9093
   
   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']
   
     - job_name: 'docker-app'
       static_configs:
         - targets: ['host.docker.internal:5000']
       metrics_path: '/metrics'
       scrape_interval: 5s
   
     - job_name: 'cadvisor'
       static_configs:
         - targets: ['cadvisor:8080']
   EOF
   ```

3. **Create AlertManager configuration:**
   ```bash
   cat > alertmanager.yml << EOF
   global:
     smtp_smarthost: 'localhost:587'
     smtp_from: 'alertmanager@example.com'
   
   route:
     group_by: ['alertname']
     group_wait: 10s
     group_interval: 10s
     repeat_interval: 1h
     receiver: 'web.hook'
   
   receivers:
   - name: 'web.hook'
     webhook_configs:
     - url: 'http://127.0.0.1:5001/'
   EOF
   ```

4. **Update Docker Compose with alerting:**
   ```bash
   cat > docker-compose-with-alerts.yml << EOF
   version: '3.8'
   
   services:
     app:
       build:
         context: .
         dockerfile: Dockerfile.monitoring
       ports:
         - "5000:5000"
       environment:
         - PYTHONUNBUFFERED=1
       deploy:
         resources:
           limits:
             cpus: '1.0'
             memory: 512M
       restart: unless-stopped
   
     prometheus:
       image: prom/prometheus:latest
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus-with-alerts.yml:/etc/prometheus/prometheus.yml
         - ./alerting-rules.yml:/etc/prometheus/alerting-rules.yml
       command:
         - '--config.file=/etc/prometheus/prometheus.yml'
         - '--storage.tsdb.path=/prometheus'
         - '--web.console.libraries=/etc/prometheus/console_libraries'
         - '--web.console.templates=/etc/prometheus/consoles'
         - '--storage.tsdb.retention.time=200h'
         - '--web.enable-lifecycle'
       restart: unless-stopped
   
     alertmanager:
       image: prom/alertmanager:latest
       ports:
         - "9093:9093"
       volumes:
         - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
       command:
         - '--config.file=/etc/alertmanager/alertmanager.yml'
         - '--storage.path=/alertmanager'
       restart: unless-stopped
   
     grafana:
       image: grafana/grafana:latest
       ports:
         - "3000:3000"
       environment:
         - GF_SECURITY_ADMIN_PASSWORD=admin
       volumes:
         - grafana-storage:/var/lib/grafana
       restart: unless-stopped
   
   volumes:
     grafana-storage:
   EOF
   ```

## 🧹 Cleanup

1. **Stop and remove containers:**
   ```bash
   docker stop monitoring-demo logging-demo profiling-demo
   docker rm monitoring-demo logging-demo profiling-demo
   ```

2. **Stop Docker Compose services:**
   ```bash
   docker-compose -f docker-compose.monitoring.yml down -v
   docker-compose -f docker-compose-with-alerts.yml down -v
   ```

3. **Remove images:**
   ```bash
   docker rmi monitoring-app logging-app profiling-app
   ```

4. **Clean up files:**
   ```bash
   rm -f monitoring-app.py enhanced-monitoring.py logging-app.py profiling-app.py \
        Dockerfile.monitoring Dockerfile.logging Dockerfile.profiling requirements.txt \
        prometheus.yml prometheus-with-alerts.yml alerting-rules.yml alertmanager.yml \
        docker-compose.monitoring.yml docker-compose-with-alerts.yml debug-tools.sh
   ```

## 📚 Key Takeaways

- **Resource Monitoring**: Use `docker stats` and Prometheus for comprehensive monitoring
- **Logging**: Implement structured logging with appropriate levels
- **Debugging**: Use container inspection and interactive debugging tools
- **Performance**: Profile applications to identify bottlenecks
- **Alerting**: Set up monitoring alerts for critical issues
- **Observability**: Combine metrics, logs, and traces for full observability

## 🔍 Troubleshooting

### Common Issues:
1. **High resource usage**: Monitor and set appropriate limits
2. **Log volume**: Implement log rotation and filtering
3. **Performance issues**: Use profiling tools to identify bottlenecks
4. **Alert noise**: Configure appropriate thresholds and grouping

### Debug Commands:
```bash
# Monitor container resources
docker stats <container>

# View container logs
docker logs <container>

# Inspect container details
docker inspect <container>

# Enter container for debugging
docker exec -it <container> /bin/bash

# Check container processes
docker exec <container> ps aux
```

## 🎯 Next Steps

- Implement production monitoring with ELK stack
- Learn about distributed tracing (Jaeger, Zipkin)
- Explore Kubernetes monitoring (Prometheus Operator)
- Study log aggregation and analysis
- Practice with real-world debugging scenarios 