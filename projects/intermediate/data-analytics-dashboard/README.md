# Project 13: Data Analytics Dashboard

A comprehensive data analytics platform with Apache Kafka, Elasticsearch, Kibana, and real-time data processing demonstrating advanced container orchestration and data pipeline management.

## 🚀 Project Overview

This project showcases a production-ready data analytics platform with:
- **Data Ingestion**: Apache Kafka for real-time data streaming
- **Data Processing**: Apache Spark for stream processing
- **Data Storage**: Elasticsearch for search and analytics
- **Visualization**: Kibana for dashboards and analytics
- **API Gateway**: Nginx for request routing
- **Monitoring**: Prometheus and Grafana for observability
- **Containerization**: Docker with multi-stage builds
- **Orchestration**: Kubernetes with Helm charts

## 📁 Project Structure

```
data-analytics-dashboard/
├── data-generator/
│   ├── src/
│   │   └── main.py
│   ├── Dockerfile
│   └── requirements.txt
├── stream-processor/
│   ├── src/
│   │   └── processor.py
│   ├── Dockerfile
│   └── requirements.txt
├── api-gateway/
│   ├── nginx.conf
│   ├── Dockerfile
│   └── docker-compose.yml
├── elasticsearch/
│   ├── config/
│   └── Dockerfile
├── kibana/
│   ├── config/
│   └── Dockerfile
├── kafka/
│   ├── config/
│   └── docker-compose.yml
├── monitoring/
│   ├── prometheus/
│   ├── grafana/
│   └── elk/
├── kubernetes/
│   ├── helm-charts/
│   ├── manifests/
│   └── monitoring/
├── scripts/
│   ├── setup.sh
│   ├── deploy.sh
│   └── test.sh
└── docs/
    ├── architecture.md
    ├── api-documentation.md
    └── deployment-guide.md
```

## 🛠️ Technology Stack

### Data Pipeline
- **Message Queue**: Apache Kafka
- **Stream Processing**: Apache Spark Streaming
- **Data Storage**: Elasticsearch
- **Visualization**: Kibana
- **Data Generation**: Python with Faker

### Infrastructure
- **Containerization**: Docker with multi-stage builds
- **Orchestration**: Kubernetes with Helm
- **API Gateway**: Nginx with Lua scripting
- **Monitoring**: Prometheus, Grafana, ELK Stack
- **CI/CD**: GitHub Actions

### Development Tools
- **Testing**: Pytest, JUnit
- **Linting**: ESLint, Prettier, Black
- **Documentation**: Swagger/OpenAPI
- **Version Control**: Git

## 🐳 Docker Setup

### Development Environment

1. **Create project structure:**
   ```bash
   mkdir data-analytics-dashboard && cd data-analytics-dashboard
   mkdir -p data-generator/src stream-processor/src api-gateway elasticsearch/config \
            kibana/config kafka/config monitoring/{prometheus,grafana,elk} \
            kubernetes/{helm-charts,manifests,monitoring} scripts docs
   ```

2. **Create data generator:**
   ```bash
   cat << EOF > data-generator/requirements.txt
   kafka-python==2.0.2
   faker==18.0.0
   requests==2.28.1
   python-dotenv==0.19.2
   EOF

   cat << EOF > data-generator/src/main.py
   import json
   import time
   import random
   from datetime import datetime
   from faker import Faker
   from kafka import KafkaProducer
   import os

   fake = Faker()

   class DataGenerator:
       def __init__(self):
           self.producer = KafkaProducer(
               bootstrap_servers=os.getenv('KAFKA_BOOTSTRAP_SERVERS', 'localhost:9092'),
               value_serializer=lambda v: json.dumps(v).encode('utf-8')
           )

       def generate_user_data(self):
           return {
               'user_id': fake.uuid4(),
               'name': fake.name(),
               'email': fake.email(),
               'age': random.randint(18, 80),
               'city': fake.city(),
               'country': fake.country(),
               'timestamp': datetime.now().isoformat(),
               'event_type': 'user_registration'
           }

       def generate_purchase_data(self):
           return {
               'purchase_id': fake.uuid4(),
               'user_id': fake.uuid4(),
               'product_name': fake.product_name(),
               'category': random.choice(['electronics', 'clothing', 'books', 'home']),
               'price': round(random.uniform(10, 1000), 2),
               'quantity': random.randint(1, 5),
               'timestamp': datetime.now().isoformat(),
               'event_type': 'purchase'
           }

       def generate_click_data(self):
           return {
               'click_id': fake.uuid4(),
               'user_id': fake.uuid4(),
               'page_url': fake.url(),
               'referrer': fake.url(),
               'user_agent': fake.user_agent(),
               'ip_address': fake.ipv4(),
               'timestamp': datetime.now().isoformat(),
               'event_type': 'page_click'
           }

       def send_data(self, topic, data):
           try:
               self.producer.send(topic, data)
               self.producer.flush()
               print(f"Sent data to {topic}: {data['event_type']}")
           except Exception as e:
               print(f"Error sending data: {e}")

       def run(self):
           topics = ['user-events', 'purchase-events', 'click-events']
           generators = [
               self.generate_user_data,
               self.generate_purchase_data,
               self.generate_click_data
           ]

           while True:
               for topic, generator in zip(topics, generators):
                   data = generator()
                   self.send_data(topic, data)
                   time.sleep(random.uniform(0.5, 2.0))

   if __name__ == '__main__':
       generator = DataGenerator()
       generator.run()
   EOF
   ```

3. **Create stream processor:**
   ```bash
   cat << EOF > stream-processor/requirements.txt
   pyspark==3.3.0
   elasticsearch==8.0.0
   kafka-python==2.0.2
   python-dotenv==0.19.2
   EOF

   cat << EOF > stream-processor/src/processor.py
   import json
   import os
   from pyspark.sql import SparkSession
   from pyspark.sql.functions import *
   from pyspark.sql.types import *
   from elasticsearch import Elasticsearch
   from kafka import KafkaConsumer
   import logging

   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)

   class StreamProcessor:
       def __init__(self):
           self.spark = SparkSession.builder \
               .appName("DataAnalyticsProcessor") \
               .config("spark.sql.adaptive.enabled", "true") \
               .getOrCreate()

           self.es_client = Elasticsearch(
               [os.getenv('ELASTICSEARCH_HOST', 'localhost:9200')],
               basic_auth=(os.getenv('ES_USER', 'elastic'), 
                          os.getenv('ES_PASSWORD', 'changeme'))
           )

           self.kafka_consumer = KafkaConsumer(
               'user-events', 'purchase-events', 'click-events',
               bootstrap_servers=os.getenv('KAFKA_BOOTSTRAP_SERVERS', 'localhost:9092'),
               value_deserializer=lambda m: json.loads(m.decode('utf-8'))
           )

       def process_user_events(self, data):
           """Process user registration events"""
           try:
               # Extract user demographics
               user_stats = {
                   'user_id': data['user_id'],
                   'age_group': self.get_age_group(data['age']),
                   'country': data['country'],
                   'city': data['city'],
                   'registration_date': data['timestamp'],
                   'event_type': 'user_registration'
               }
               
               # Index to Elasticsearch
               self.es_client.index(
                   index='user-analytics',
                   document=user_stats
               )
               logger.info(f"Processed user event: {data['user_id']}")
           except Exception as e:
               logger.error(f"Error processing user event: {e}")

       def process_purchase_events(self, data):
           """Process purchase events"""
           try:
               # Calculate purchase metrics
               purchase_stats = {
                   'purchase_id': data['purchase_id'],
                   'user_id': data['user_id'],
                   'product_category': data['category'],
                   'total_amount': data['price'] * data['quantity'],
                   'quantity': data['quantity'],
                   'purchase_date': data['timestamp'],
                   'event_type': 'purchase'
               }
               
               # Index to Elasticsearch
               self.es_client.index(
                   index='purchase-analytics',
                   document=purchase_stats
               )
               logger.info(f"Processed purchase event: {data['purchase_id']}")
           except Exception as e:
               logger.error(f"Error processing purchase event: {e}")

       def process_click_events(self, data):
           """Process click events"""
           try:
               # Extract click analytics
               click_stats = {
                   'click_id': data['click_id'],
                   'user_id': data['user_id'],
                   'page_url': data['page_url'],
                   'referrer': data['referrer'],
                   'ip_address': data['ip_address'],
                   'click_date': data['timestamp'],
                   'event_type': 'page_click'
               }
               
               # Index to Elasticsearch
               self.es_client.index(
                   index='click-analytics',
                   document=click_stats
               )
               logger.info(f"Processed click event: {data['click_id']}")
           except Exception as e:
               logger.error(f"Error processing click event: {e}")

       def get_age_group(self, age):
           """Categorize age into groups"""
           if age < 25:
               return '18-24'
           elif age < 35:
               return '25-34'
           elif age < 45:
               return '35-44'
           elif age < 55:
               return '45-54'
           else:
               return '55+'

       def run(self):
           """Main processing loop"""
           logger.info("Starting stream processor...")
           
           for message in self.kafka_consumer:
               try:
                   data = message.value
                   event_type = data.get('event_type', '')
                   
                   if event_type == 'user_registration':
                       self.process_user_events(data)
                   elif event_type == 'purchase':
                       self.process_purchase_events(data)
                   elif event_type == 'page_click':
                       self.process_click_events(data)
                   
               except Exception as e:
                   logger.error(f"Error processing message: {e}")

   if __name__ == '__main__':
       processor = StreamProcessor()
       processor.run()
   EOF
   ```

4. **Create API Gateway:**
   ```bash
   cat << EOF > api-gateway/nginx.conf
   events {
       worker_connections 1024;
   }

   http {
       upstream elasticsearch {
           server elasticsearch:9200;
       }

       upstream kibana {
           server kibana:5601;
       }

       server {
           listen 80;
           server_name analytics.local;

           # API endpoints
           location /api/analytics/ {
               proxy_pass http://elasticsearch/_search;
               proxy_set_header Host \$host;
               proxy_set_header X-Real-IP \$remote_addr;
               proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
           }

           # Kibana dashboard
           location /dashboard/ {
               proxy_pass http://kibana/;
               proxy_set_header Host \$host;
               proxy_set_header X-Real-IP \$remote_addr;
               proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
           }

           # Health check
           location /health {
               return 200 "healthy";
               add_header Content-Type text/plain;
           }
       }
   }
   EOF
   ```

5. **Create Docker Compose for development:**
   ```bash
   cat << EOF > docker-compose.yml
   version: '3.8'

   services:
     zookeeper:
       image: confluentinc/cp-zookeeper:latest
       environment:
         ZOOKEEPER_CLIENT_PORT: 2181
         ZOOKEEPER_TICK_TIME: 2000
       ports:
         - "2181:2181"

     kafka:
       image: confluentinc/cp-kafka:latest
       depends_on:
         - zookeeper
       ports:
         - "9092:9092"
       environment:
         KAFKA_BROKER_ID: 1
         KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
         KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
         KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

     elasticsearch:
       image: docker.elastic.co/elasticsearch/elasticsearch:8.0.0
       environment:
         - discovery.type=single-node
         - xpack.security.enabled=false
         - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
       ports:
         - "9200:9200"
       volumes:
         - elasticsearch_data:/usr/share/elasticsearch/data

     kibana:
       image: docker.elastic.co/kibana/kibana:8.0.0
       environment:
         - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
       ports:
         - "5601:5601"
       depends_on:
         - elasticsearch

     data-generator:
       build:
         context: ./data-generator
         dockerfile: Dockerfile
       environment:
         - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
       depends_on:
         - kafka
       restart: unless-stopped

     stream-processor:
       build:
         context: ./stream-processor
         dockerfile: Dockerfile
       environment:
         - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
         - ELASTICSEARCH_HOST=elasticsearch:9200
       depends_on:
         - kafka
         - elasticsearch
       restart: unless-stopped

     api-gateway:
       image: nginx:alpine
       ports:
         - "80:80"
       volumes:
         - ./api-gateway/nginx.conf:/etc/nginx/nginx.conf
       depends_on:
         - elasticsearch
         - kibana
       restart: unless-stopped

     prometheus:
       image: prom/prometheus:latest
       ports:
         - "9090:9090"
       volumes:
         - ./monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
       command:
         - '--config.file=/etc/prometheus/prometheus.yml'

     grafana:
       image: grafana/grafana:latest
       ports:
         - "3000:3000"
       environment:
         - GF_SECURITY_ADMIN_PASSWORD=admin
       volumes:
         - grafana_data:/var/lib/grafana

   volumes:
     elasticsearch_data:
     grafana_data:
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
     name: data-analytics
   EOF
   ```

2. **Kafka StatefulSet:**
   ```bash
   cat << EOF > kubernetes/kafka-statefulset.yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: kafka
     namespace: data-analytics
   spec:
     serviceName: kafka-service
     replicas: 3
     selector:
       matchLabels:
         app: kafka
     template:
       metadata:
         labels:
           app: kafka
       spec:
         containers:
         - name: kafka
           image: confluentinc/cp-kafka:latest
           ports:
           - containerPort: 9092
           env:
           - name: KAFKA_BROKER_ID
             valueFrom:
               fieldRef:
                 fieldPath: metadata.ordinal
           - name: KAFKA_ZOOKEEPER_CONNECT
             value: "zookeeper:2181"
           - name: KAFKA_ADVERTISED_LISTENERS
             value: "PLAINTEXT://kafka-\${KAFKA_BROKER_ID}.kafka-service:9092"
           - name: KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR
             value: "3"
           volumeMounts:
           - name: kafka-storage
             mountPath: /var/lib/kafka/data
     volumeClaimTemplates:
     - metadata:
         name: kafka-storage
       spec:
         accessModes: ["ReadWriteOnce"]
         resources:
           requests:
             storage: 10Gi
   EOF
   ```

3. **Elasticsearch StatefulSet:**
   ```bash
   cat << EOF > kubernetes/elasticsearch-statefulset.yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: elasticsearch
     namespace: data-analytics
   spec:
     serviceName: elasticsearch-service
     replicas: 3
     selector:
       matchLabels:
         app: elasticsearch
     template:
       metadata:
         labels:
           app: elasticsearch
       spec:
         containers:
         - name: elasticsearch
           image: docker.elastic.co/elasticsearch/elasticsearch:8.0.0
           ports:
           - containerPort: 9200
           env:
           - name: discovery.type
             value: "single-node"
           - name: xpack.security.enabled
             value: "false"
           - name: ES_JAVA_OPTS
             value: "-Xms1g -Xmx1g"
           volumeMounts:
           - name: elasticsearch-storage
             mountPath: /usr/share/elasticsearch/data
           resources:
             requests:
               memory: "1Gi"
               cpu: "500m"
             limits:
               memory: "2Gi"
               cpu: "1000m"
     volumeClaimTemplates:
     - metadata:
         name: elasticsearch-storage
       spec:
         accessModes: ["ReadWriteOnce"]
         resources:
           requests:
             storage: 20Gi
   EOF
   ```

4. **Kibana Deployment:**
   ```bash
   cat << EOF > kubernetes/kibana-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kibana
     namespace: data-analytics
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kibana
     template:
       metadata:
         labels:
           app: kibana
       spec:
         containers:
         - name: kibana
           image: docker.elastic.co/kibana/kibana:8.0.0
           ports:
           - containerPort: 5601
           env:
           - name: ELASTICSEARCH_HOSTS
             value: "http://elasticsearch-service:9200"
           resources:
             requests:
               memory: "512Mi"
               cpu: "250m"
             limits:
               memory: "1Gi"
               cpu: "500m"
   EOF
   ```

5. **Data Generator Deployment:**
   ```bash
   cat << EOF > kubernetes/data-generator-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: data-generator
     namespace: data-analytics
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: data-generator
     template:
       metadata:
         labels:
           app: data-generator
       spec:
         containers:
         - name: data-generator
           image: data-generator:latest
           env:
           - name: KAFKA_BOOTSTRAP_SERVERS
             value: "kafka-service:9092"
           resources:
             requests:
               memory: "256Mi"
               cpu: "200m"
             limits:
               memory: "512Mi"
               cpu: "500m"
   EOF
   ```

6. **Stream Processor Deployment:**
   ```bash
   cat << EOF > kubernetes/stream-processor-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: stream-processor
     namespace: data-analytics
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: stream-processor
     template:
       metadata:
         labels:
           app: stream-processor
       spec:
         containers:
         - name: stream-processor
           image: stream-processor:latest
           env:
           - name: KAFKA_BOOTSTRAP_SERVERS
             value: "kafka-service:9092"
           - name: ELASTICSEARCH_HOST
             value: "elasticsearch-service:9200"
           resources:
             requests:
               memory: "1Gi"
               cpu: "500m"
             limits:
               memory: "2Gi"
               cpu: "1000m"
   EOF
   ```

7. **Services:**
   ```bash
   cat << EOF > kubernetes/services.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: kafka-service
     namespace: data-analytics
   spec:
     clusterIP: None
     selector:
       app: kafka
     ports:
     - port: 9092
       targetPort: 9092
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: elasticsearch-service
     namespace: data-analytics
   spec:
     selector:
       app: elasticsearch
     ports:
     - port: 9200
       targetPort: 9200
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: kibana-service
     namespace: data-analytics
   spec:
     selector:
       app: kibana
     ports:
     - port: 5601
       targetPort: 5601
   EOF
   ```

8. **Ingress:**
   ```bash
   cat << EOF > kubernetes/ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: analytics-ingress
     namespace: data-analytics
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     ingressClassName: nginx
     rules:
     - host: analytics.local
       http:
         paths:
         - path: /kibana
           pathType: Prefix
           backend:
             service:
               name: kibana-service
               port:
                 number: 5601
         - path: /api
           pathType: Prefix
           backend:
             service:
               name: elasticsearch-service
               port:
                 number: 9200
   EOF
   ```

## 🚀 Deployment

### Docker Deployment
```bash
# Start the entire stack
docker-compose up -d

# Check services
docker-compose ps

# View logs
docker-compose logs -f data-generator
docker-compose logs -f stream-processor
```

### Kubernetes Deployment
```bash
# Apply all manifests
kubectl apply -f kubernetes/

# Check deployment status
kubectl get pods -n data-analytics
kubectl get services -n data-analytics

# Port forward to access services
kubectl port-forward -n data-analytics svc/kibana-service 5601:5601
kubectl port-forward -n data-analytics svc/elasticsearch-service 9200:9200
```

## 📊 Monitoring

### Kibana Dashboards
- Access Kibana at `http://localhost:5601`
- Create dashboards for:
  - User registration analytics
  - Purchase trends
  - Click stream analysis
  - Geographic distribution

### Elasticsearch Queries
```bash
# Get user analytics
curl -X GET "localhost:9200/user-analytics/_search"

# Get purchase analytics
curl -X GET "localhost:9200/purchase-analytics/_search"

# Get click analytics
curl -X GET "localhost:9200/click-analytics/_search"
```

## 🧪 Testing

### Data Generation Test
```bash
# Check if data is being generated
kubectl logs -n data-analytics -l app=data-generator

# Check if data is being processed
kubectl logs -n data-analytics -l app=stream-processor
```

### Elasticsearch Test
```bash
# Test Elasticsearch connectivity
curl -X GET "localhost:9200/_cluster/health"

# Test data indexing
curl -X GET "localhost:9200/user-analytics/_count"
```

## 🧹 Cleanup

### Docker Cleanup
```bash
docker-compose down -v
```

### Kubernetes Cleanup
```bash
kubectl delete namespace data-analytics
```

## 🎯 Learning Objectives
- Real-time data streaming with Kafka
- Stream processing with Apache Spark
- Data storage and search with Elasticsearch
- Data visualization with Kibana
- Multi-service orchestration
- Data pipeline monitoring

## 📚 Key Takeaways
- **Data Streaming**: Kafka for real-time data ingestion
- **Stream Processing**: Apache Spark for data transformation
- **Search & Analytics**: Elasticsearch for data storage and search
- **Visualization**: Kibana for dashboard creation
- **Scalability**: Horizontal scaling of data processing
- **Monitoring**: End-to-end data pipeline observability

---

**Happy Learning! 📊🐳☸️** 