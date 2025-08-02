# Microservices Application - Intermediate Project

A comprehensive microservices application demonstrating Docker and Kubernetes concepts with multiple services, databases, and monitoring.

## 🚀 Project Overview

This project showcases a production-ready microservices application with:
- **API Gateway**: Nginx reverse proxy with load balancing
- **User Service**: Node.js service for user management
- **Product Service**: Node.js service for product catalog
- **Order Service**: Node.js service for order processing
- **Database**: PostgreSQL for user and product data
- **Cache**: Redis for session and caching
- **Message Queue**: RabbitMQ for service communication
- **Monitoring**: Prometheus, Grafana, and ELK Stack
- **Containerization**: Docker with multi-stage builds
- **Orchestration**: Kubernetes with Helm charts

## 📁 Project Structure

```
microservices-app/
├── services/
│   ├── api-gateway/          # Nginx reverse proxy
│   ├── user-service/         # User management service
│   ├── product-service/      # Product catalog service
│   └── order-service/        # Order processing service
├── databases/
│   ├── postgres/             # Database configurations
│   └── redis/                # Cache configurations
├── message-queue/
│   └── rabbitmq/             # Message queue setup
├── monitoring/
│   ├── prometheus/           # Metrics collection
│   ├── grafana/              # Visualization dashboards
│   └── elk/                  # Logging stack
├── docker/
│   ├── docker-compose.yml    # Development environment
│   ├── docker-compose.prod.yml # Production environment
│   └── nginx/
├── kubernetes/
│   ├── helm-charts/          # Helm charts for deployment
│   ├── manifests/            # Kubernetes manifests
│   └── monitoring/           # K8s monitoring stack
├── scripts/
│   ├── setup.sh              # Environment setup
│   ├── deploy.sh             # Deployment scripts
│   └── test.sh               # Testing scripts
└── docs/
    ├── architecture.md        # System architecture
    ├── api-documentation.md  # API documentation
    └── deployment-guide.md   # Deployment instructions
```

## 🛠️ Technology Stack

### Services
- **API Gateway**: Nginx with Lua scripting
- **User Service**: Node.js with Express, JWT authentication
- **Product Service**: Node.js with Express, caching
- **Order Service**: Node.js with Express, event-driven

### Infrastructure
- **Databases**: PostgreSQL (primary), Redis (cache)
- **Message Queue**: RabbitMQ for async communication
- **Monitoring**: Prometheus, Grafana, ELK Stack
- **Containerization**: Docker with multi-stage builds
- **Orchestration**: Kubernetes with Helm
- **CI/CD**: GitHub Actions

### Development Tools
- **Testing**: Jest, Supertest, Cypress
- **Linting**: ESLint, Prettier
- **Type Checking**: TypeScript
- **Documentation**: Swagger/OpenAPI

## 🐳 Docker Setup

### Development Environment

1. **Clone and setup:**
   ```bash
   git clone <repository-url>
   cd microservices-app
   chmod +x scripts/setup.sh
   ./scripts/setup.sh
   ```

2. **Start development environment:**
   ```bash
   docker-compose up -d
   ```

3. **Verify services:**
   ```bash
   # Check all services are running
   docker-compose ps
   
   # Check service logs
   docker-compose logs -f
   ```

4. **Access services:**
   - API Gateway: http://localhost:8080
   - User Service: http://localhost:3001
   - Product Service: http://localhost:3002
   - Order Service: http://localhost:3003
   - Grafana: http://localhost:3000
   - Kibana: http://localhost:5601

### Production Environment

1. **Build production images:**
   ```bash
   docker-compose -f docker-compose.prod.yml build
   ```

2. **Deploy to production:**
   ```bash
   docker-compose -f docker-compose.prod.yml up -d
   ```

## ☸️ Kubernetes Deployment

### Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured
- Helm 3.x installed

### Deploy with Helm

1. **Add Helm repositories:**
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```

2. **Deploy infrastructure:**
   ```bash
   # Deploy PostgreSQL
   helm install postgres bitnami/postgresql \
     --set auth.postgresPassword=password \
     --set auth.database=microservices

   # Deploy Redis
   helm install redis bitnami/redis \
     --set auth.enabled=false

   # Deploy RabbitMQ
   helm install rabbitmq bitnami/rabbitmq \
     --set auth.username=admin \
     --set auth.password=password
   ```

3. **Deploy application services:**
   ```bash
   # Deploy all services
   helm install microservices ./kubernetes/helm-charts/microservices
   ```

4. **Deploy monitoring stack:**
   ```bash
   # Deploy Prometheus and Grafana
   helm install monitoring prometheus-community/kube-prometheus-stack
   ```

### Deploy with Manifests

1. **Create namespace:**
   ```bash
   kubectl apply -f kubernetes/manifests/namespace.yaml
   ```

2. **Deploy infrastructure:**
   ```bash
   kubectl apply -f kubernetes/manifests/infrastructure/
   ```

3. **Deploy services:**
   ```bash
   kubectl apply -f kubernetes/manifests/services/
   ```

4. **Deploy monitoring:**
   ```bash
   kubectl apply -f kubernetes/monitoring/
   ```

## 📊 Monitoring and Observability

### Metrics Collection
- **Prometheus**: Collects metrics from all services
- **Custom Metrics**: Business metrics for orders, users, products
- **Infrastructure Metrics**: CPU, memory, disk usage

### Visualization
- **Grafana Dashboards**: Pre-configured dashboards for:
  - Service performance
  - Business metrics
  - Infrastructure monitoring
  - Error rates and latency

### Logging
- **ELK Stack**: Centralized logging with:
  - Structured logging from all services
  - Log aggregation and search
  - Error tracking and alerting

### Alerting
- **Prometheus Alerts**: Configured for:
  - High error rates
  - Service unavailability
  - Resource usage thresholds
  - Business metric anomalies

## 🧪 Testing

### Unit Tests
```bash
# Run unit tests for all services
docker-compose exec user-service npm test
docker-compose exec product-service npm test
docker-compose exec order-service npm test
```

### Integration Tests
```bash
# Run integration tests
./scripts/test.sh integration
```

### Load Testing
```bash
# Run load tests
docker-compose exec k6 k6 run /scripts/load-test.js
```

### End-to-End Tests
```bash
# Run E2E tests
docker-compose exec cypress npm run test:e2e
```

## 🔒 Security

### Security Features
- **JWT Authentication**: Secure user authentication
- **API Rate Limiting**: Prevent abuse
- **Input Validation**: Sanitize all inputs
- **SQL Injection Prevention**: Parameterized queries
- **XSS Protection**: Content Security Policy
- **HTTPS**: TLS encryption for all communications
- **Secrets Management**: Kubernetes secrets for sensitive data

### Security Scanning
```bash
# Scan Docker images
docker scan microservices-user-service:latest
docker scan microservices-product-service:latest
docker scan microservices-order-service:latest

# Run security audit
npm audit
```

## 📈 Performance

### Optimization Features
- **Caching**: Redis for frequently accessed data
- **Database Optimization**: Indexes and query optimization
- **Load Balancing**: Nginx load balancer
- **Horizontal Scaling**: Kubernetes HPA
- **CDN Integration**: Static asset delivery
- **Connection Pooling**: Database connection optimization

### Performance Monitoring
- **Response Time Tracking**: APM with custom metrics
- **Throughput Monitoring**: Requests per second
- **Error Rate Tracking**: Failed request percentage
- **Resource Utilization**: CPU, memory, disk usage

## 🔧 Development Workflow

### Local Development
1. **Start development environment**
2. **Make code changes**
3. **Run tests**
4. **Commit changes**
5. **Push to repository**

### CI/CD Pipeline
1. **Code push triggers pipeline**
2. **Run tests and security scans**
3. **Build Docker images**
4. **Deploy to staging**
5. **Run integration tests**
6. **Deploy to production**

## 🚨 Troubleshooting

### Common Issues

1. **Service communication issues:**
   ```bash
   # Check service health
   curl http://localhost:3001/health
   curl http://localhost:3002/health
   curl http://localhost:3003/health
   
   # Check RabbitMQ connection
   docker-compose exec rabbitmq rabbitmqctl list_connections
   ```

2. **Database connection issues:**
   ```bash
   # Check PostgreSQL
   docker-compose exec postgres psql -U postgres -d microservices
   
   # Check Redis
   docker-compose exec redis redis-cli ping
   ```

3. **Monitoring issues:**
   ```bash
   # Check Prometheus targets
   curl http://localhost:9090/api/v1/targets
   
   # Check Grafana
   curl http://localhost:3000/api/health
   ```

### Debug Commands
```bash
# Check service status
docker-compose ps

# View service logs
docker-compose logs -f [service-name]

# Execute commands in containers
docker-compose exec [service-name] [command]

# Check resource usage
docker stats
```

## 📚 Learning Objectives

This project demonstrates:

### Microservices Architecture
- ✅ Service decomposition
- ✅ API gateway patterns
- ✅ Service communication
- ✅ Data consistency
- ✅ Event-driven architecture

### Docker Advanced Concepts
- ✅ Multi-stage builds
- ✅ Service orchestration
- ✅ Network configuration
- ✅ Volume management
- ✅ Health checks

### Kubernetes Advanced Features
- ✅ Helm charts
- ✅ Service mesh concepts
- ✅ Advanced networking
- ✅ Resource management
- ✅ Monitoring integration

### DevOps Practices
- ✅ CI/CD pipelines
- ✅ Infrastructure as Code
- ✅ Monitoring and alerting
- ✅ Security best practices
- ✅ Performance optimization

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new features
5. Ensure all tests pass
6. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👨‍💻 Author

**Olúmáyòwá Akinkuehinmi**
- 📧 Email: akintunero101@gmail.com
- 🐦 Twitter: [@akintunero](https://twitter.com/akintunero)
- 💼 LinkedIn: [olumayowaa](https://linkedin.com/in/olumayowaa)

---

**Happy Learning! 🚀** 