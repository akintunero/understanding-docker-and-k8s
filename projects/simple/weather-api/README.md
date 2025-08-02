# Project 03: Weather API

A Python Flask API service with Redis caching demonstrating environment management and Kubernetes services.

## 🚀 Project Overview

This project showcases a weather API service with:
- **Backend**: Python Flask API
- **Cache**: Redis for response caching
- **External API**: OpenWeatherMap integration
- **Containerization**: Docker with environment management
- **Orchestration**: Kubernetes with services and ingress
- **Monitoring**: Basic health checks and metrics

## 📁 Project Structure

```
weather-api/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── weather.py
│   └── cache.py
├── tests/
│   ├── test_weather.py
│   └── test_cache.py
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── kubernetes/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── requirements.txt
├── .env.example
└── README.md
```

## 🛠️ Technology Stack

### Backend
- **Framework**: Flask
- **Language**: Python 3.9
- **Cache**: Redis
- **HTTP Client**: Requests
- **Validation**: Pydantic
- **Testing**: Pytest

### Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **API Gateway**: Nginx Ingress
- **Monitoring**: Prometheus metrics
- **Logging**: Structured logging

## 🐳 Docker Setup

### Development Environment

1. **Create project structure:**
   ```bash
   mkdir weather-api && cd weather-api
   mkdir -p app tests docker kubernetes
   ```

2. **Create Flask application:**
   ```bash
   # Create requirements.txt
   cat << EOF > requirements.txt
   flask==2.0.1
   redis==4.0.2
   requests==2.26.0
   pydantic==1.8.2
   prometheus-client==0.11.0
   python-dotenv==0.19.0
   pytest==6.2.5
   EOF

   # Create main application
   cat << EOF > app/main.py
   from flask import Flask, jsonify, request
   from app.weather import WeatherService
   from app.cache import RedisCache
   import os
   import logging

   app = Flask(__name__)
   weather_service = WeatherService()
   cache = RedisCache()

   @app.route('/health')
   def health():
       return jsonify({'status': 'healthy'})

   @app.route('/weather/<city>')
   def get_weather(city):
       try:
           # Check cache first
           cached_data = cache.get(f"weather:{city}")
           if cached_data:
               return jsonify(cached_data)

           # Get weather data
           weather_data = weather_service.get_weather(city)
           
           # Cache the result
           cache.set(f"weather:{city}", weather_data, ttl=300)
           
           return jsonify(weather_data)
       except Exception as e:
           return jsonify({'error': str(e)}), 500

   @app.route('/metrics')
   def metrics():
       return jsonify({
           'cache_hits': cache.get_stats().get('hits', 0),
           'cache_misses': cache.get_stats().get('misses', 0)
       })

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

3. **Create weather service:**
   ```bash
   cat << EOF > app/weather.py
   import requests
   import os
   from typing import Dict, Any

   class WeatherService:
       def __init__(self):
           self.api_key = os.getenv('OPENWEATHER_API_KEY', 'demo-key')
           self.base_url = 'http://api.openweathermap.org/data/2.5/weather'

       def get_weather(self, city: str) -> Dict[str, Any]:
           params = {
               'q': city,
               'appid': self.api_key,
               'units': 'metric'
           }
           
           response = requests.get(self.base_url, params=params)
           response.raise_for_status()
           
           data = response.json()
           return {
               'city': data['name'],
               'country': data['sys']['country'],
               'temperature': data['main']['temp'],
               'description': data['weather'][0]['description'],
               'humidity': data['main']['humidity'],
               'wind_speed': data['wind']['speed']
           }
   EOF
   ```

4. **Create cache service:**
   ```bash
   cat << EOF > app/cache.py
   import redis
   import json
   import os
   from typing import Any, Optional

   class RedisCache:
       def __init__(self):
           self.redis_client = redis.Redis(
               host=os.getenv('REDIS_HOST', 'localhost'),
               port=int(os.getenv('REDIS_PORT', 6379)),
               decode_responses=True
           )
           self.stats = {'hits': 0, 'misses': 0}

       def get(self, key: str) -> Optional[Any]:
           try:
               data = self.redis_client.get(key)
               if data:
                   self.stats['hits'] += 1
                   return json.loads(data)
               else:
                   self.stats['misses'] += 1
                   return None
           except Exception:
               return None

       def set(self, key: str, value: Any, ttl: int = 300) -> bool:
           try:
               self.redis_client.setex(key, ttl, json.dumps(value))
               return True
           except Exception:
               return False

       def get_stats(self) -> dict:
           return self.stats
   EOF
   ```

5. **Create Dockerfile:**
   ```bash
   cat << EOF > docker/Dockerfile
   FROM python:3.9-slim

   WORKDIR /app

   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt

   COPY app/ ./app/

   EXPOSE 5000

   CMD ["python", "-m", "flask", "run", "--host=0.0.0.0", "--port=5000"]
   EOF
   ```

6. **Create docker-compose.yml:**
   ```bash
   cat << EOF > docker/docker-compose.yml
   version: '3.8'

   services:
     weather-api:
       build:
         context: ..
         dockerfile: docker/Dockerfile
       ports:
         - "5000:5000"
       environment:
         - FLASK_ENV=development
         - REDIS_HOST=redis
         - OPENWEATHER_API_KEY=your-api-key
       depends_on:
         - redis
       restart: unless-stopped

     redis:
       image: redis:alpine
       ports:
         - "6379:6379"
       volumes:
         - redis_data:/data
       restart: unless-stopped

   volumes:
     redis_data:
   EOF
   ```

## ☸️ Kubernetes Setup

### Create Kubernetes manifests

1. **Namespace:**
   ```bash
   cat << EOF > kubernetes/namespace.yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: weather-api
   EOF
   ```

2. **ConfigMap:**
   ```bash
   cat << EOF > kubernetes/configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: weather-api-config
     namespace: weather-api
   data:
     REDIS_HOST: redis-service
     REDIS_PORT: "6379"
     FLASK_ENV: production
   EOF
   ```

3. **Secret:**
   ```bash
   cat << EOF > kubernetes/secret.yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: weather-api-secrets
     namespace: weather-api
   type: Opaque
   data:
     OPENWEATHER_API_KEY: <base64-encoded-api-key>
   EOF
   ```

4. **Redis Deployment:**
   ```bash
   cat << EOF > kubernetes/redis-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: redis
     namespace: weather-api
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: redis
     template:
       metadata:
         labels:
           app: redis
       spec:
         containers:
         - name: redis
           image: redis:alpine
           ports:
           - containerPort: 6379
           resources:
             requests:
               memory: "64Mi"
               cpu: "100m"
             limits:
               memory: "128Mi"
               cpu: "200m"
   EOF
   ```

5. **Redis Service:**
   ```bash
   cat << EOF > kubernetes/redis-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: redis-service
     namespace: weather-api
   spec:
     selector:
       app: redis
     ports:
     - port: 6379
       targetPort: 6379
     type: ClusterIP
   EOF
   ```

6. **Weather API Deployment:**
   ```bash
   cat << EOF > kubernetes/deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: weather-api
     namespace: weather-api
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: weather-api
     template:
       metadata:
         labels:
           app: weather-api
       spec:
         containers:
         - name: weather-api
           image: weather-api:latest
           ports:
           - containerPort: 5000
           env:
           - name: REDIS_HOST
             valueFrom:
               configMapKeyRef:
                 name: weather-api-config
                 key: REDIS_HOST
           - name: REDIS_PORT
             valueFrom:
               configMapKeyRef:
                 name: weather-api-config
                 key: REDIS_PORT
           - name: OPENWEATHER_API_KEY
             valueFrom:
               secretKeyRef:
                 name: weather-api-secrets
                 key: OPENWEATHER_API_KEY
           resources:
             requests:
               memory: "128Mi"
               cpu: "200m"
             limits:
               memory: "256Mi"
               cpu: "500m"
           livenessProbe:
             httpGet:
               path: /health
               port: 5000
             initialDelaySeconds: 30
             periodSeconds: 10
           readinessProbe:
             httpGet:
               path: /health
               port: 5000
             initialDelaySeconds: 5
             periodSeconds: 5
   EOF
   ```

7. **Weather API Service:**
   ```bash
   cat << EOF > kubernetes/service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: weather-api-service
     namespace: weather-api
   spec:
     selector:
       app: weather-api
     ports:
     - port: 80
       targetPort: 5000
     type: ClusterIP
   EOF
   ```

8. **Ingress:**
   ```bash
   cat << EOF > kubernetes/ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: weather-api-ingress
     namespace: weather-api
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     ingressClassName: nginx
     rules:
     - host: weather-api.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: weather-api-service
               port:
                 number: 80
   EOF
   ```

## 🚀 Deployment

### Docker Deployment
```bash
# Build and run with Docker Compose
cd docker
docker-compose up -d

# Test the API
curl http://localhost:5000/health
curl http://localhost:5000/weather/london
```

### Kubernetes Deployment
```bash
# Apply all manifests
kubectl apply -f kubernetes/

# Check deployment status
kubectl get pods -n weather-api
kubectl get services -n weather-api

# Port forward to test
kubectl port-forward -n weather-api svc/weather-api-service 8080:80

# Test the API
curl http://localhost:8080/health
curl http://localhost:8080/weather/paris
```

## 🧪 Testing

### Unit Tests
```bash
# Create test file
cat << EOF > tests/test_weather.py
import pytest
from app.weather import WeatherService
from app.cache import RedisCache

def test_weather_service():
    service = WeatherService()
    # Add test cases here
    assert True

def test_cache():
    cache = RedisCache()
    # Add test cases here
    assert True
EOF

# Run tests
pytest tests/
```

## 📊 Monitoring

### Health Checks
- `/health` - Application health status
- `/metrics` - Cache statistics

### Logging
```bash
# View application logs
kubectl logs -n weather-api -l app=weather-api

# View Redis logs
kubectl logs -n weather-api -l app=redis
```

## 🧹 Cleanup

### Docker Cleanup
```bash
cd docker
docker-compose down -v
```

### Kubernetes Cleanup
```bash
kubectl delete namespace weather-api
```

## 🎯 Learning Objectives
- Environment variable management in containers
- Service communication (API ↔ Redis)
- Kubernetes service discovery
- Health checks and monitoring
- Caching strategies
- API design and documentation

## 📚 Key Takeaways
- **Environment Management**: Use ConfigMaps and Secrets for configuration
- **Service Communication**: Redis caching for performance
- **Health Checks**: Implement proper health endpoints
- **Monitoring**: Basic metrics and logging
- **Caching**: Redis for response caching
- **API Design**: RESTful API with proper error handling

---

**Happy Learning! 🌤️🐳☸️** 