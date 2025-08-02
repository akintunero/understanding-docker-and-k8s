# Todo Application - Docker & Kubernetes Project

A complete todo application demonstrating Docker and Kubernetes concepts with a full-stack implementation.

## 🚀 Project Overview

This project showcases a production-ready todo application with:
- **Frontend**: React.js with TypeScript
- **Backend**: Node.js with Express
- **Database**: PostgreSQL
- **Cache**: Redis
- **Containerization**: Docker with multi-stage builds
- **Orchestration**: Kubernetes deployment
- **Monitoring**: Prometheus and Grafana
- **Logging**: ELK Stack

## 📁 Project Structure

```
todo-app/
├── frontend/                 # React.js frontend
│   ├── src/
│   ├── public/
│   ├── Dockerfile
│   └── package.json
├── backend/                  # Node.js backend
│   ├── src/
│   ├── Dockerfile
│   └── package.json
├── database/                 # Database migrations
│   ├── migrations/
│   └── init.sql
├── docker/                   # Docker configurations
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   └── nginx/
├── kubernetes/               # Kubernetes manifests
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── monitoring/
├── monitoring/               # Monitoring stack
│   ├── prometheus/
│   ├── grafana/
│   └── elk/
└── README.md
```

## 🛠️ Technology Stack

### Frontend
- **Framework**: React 18 with TypeScript
- **Styling**: Tailwind CSS
- **State Management**: Redux Toolkit
- **HTTP Client**: Axios
- **Testing**: Jest + React Testing Library

### Backend
- **Runtime**: Node.js 18
- **Framework**: Express.js
- **Database**: PostgreSQL with Prisma ORM
- **Cache**: Redis
- **Authentication**: JWT
- **Validation**: Joi
- **Testing**: Jest + Supertest

### Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **Reverse Proxy**: Nginx
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **CI/CD**: GitHub Actions

## 🐳 Docker Setup

### Development Environment

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd todo-app
   ```

2. **Start the development environment:**
   ```bash
   # Start all services
   docker-compose up -d
   
   # View logs
   docker-compose logs -f
   
   # Stop services
   docker-compose down
   ```

3. **Access the application:**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8000
   - Database: localhost:5432
   - Redis: localhost:6379

### Production Environment

1. **Build production images:**
   ```bash
   # Build all services
   docker-compose -f docker-compose.prod.yml build
   
   # Start production stack
   docker-compose -f docker-compose.prod.yml up -d
   ```

2. **Environment variables:**
   ```bash
   # Copy environment template
   cp .env.example .env
   
   # Edit environment variables
   nano .env
   ```

## ☸️ Kubernetes Deployment

### Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured
- Helm (optional)

### Deploy to Kubernetes

1. **Create namespace:**
   ```bash
   kubectl apply -f kubernetes/namespace.yaml
   ```

2. **Create secrets and configmaps:**
   ```bash
   kubectl apply -f kubernetes/secret.yaml
   kubectl apply -f kubernetes/configmap.yaml
   ```

3. **Deploy the application:**
   ```bash
   kubectl apply -f kubernetes/deployment.yaml
   kubectl apply -f kubernetes/service.yaml
   kubectl apply -f kubernetes/ingress.yaml
   ```

4. **Deploy monitoring stack:**
   ```bash
   kubectl apply -f kubernetes/monitoring/
   ```

### Access the Application

```bash
# Port forward to access services
kubectl port-forward svc/todo-frontend 3000:80
kubectl port-forward svc/todo-backend 8000:80

# Access via ingress (if configured)
kubectl get ingress
```

## 📊 Monitoring & Observability

### Prometheus Metrics
- Application metrics available at `/metrics`
- Custom business metrics
- Performance monitoring

### Grafana Dashboards
- Application performance
- Database metrics
- Infrastructure monitoring
- Custom business dashboards

### ELK Stack
- Centralized logging
- Log analysis and visualization
- Error tracking and alerting

## 🔧 Development Workflow

### Local Development

1. **Start development environment:**
   ```bash
   docker-compose up -d
   ```

2. **Run database migrations:**
   ```bash
   docker-compose exec backend npm run migrate
   ```

3. **Seed database:**
   ```bash
   docker-compose exec backend npm run seed
   ```

4. **Run tests:**
   ```bash
   # Frontend tests
   docker-compose exec frontend npm test
   
   # Backend tests
   docker-compose exec backend npm test
   ```

### Code Quality

1. **Linting:**
   ```bash
   # Frontend
   docker-compose exec frontend npm run lint
   
   # Backend
   docker-compose exec backend npm run lint
   ```

2. **Type checking:**
   ```bash
   # Frontend
   docker-compose exec frontend npm run type-check
   
   # Backend
   docker-compose exec backend npm run type-check
   ```

## 🧪 Testing

### Unit Tests
```bash
# Frontend unit tests
docker-compose exec frontend npm test

# Backend unit tests
docker-compose exec backend npm test
```

### Integration Tests
```bash
# Run integration tests
docker-compose exec backend npm run test:integration
```

### E2E Tests
```bash
# Run end-to-end tests
docker-compose exec frontend npm run test:e2e
```

## 🔒 Security

### Security Features
- JWT authentication
- Input validation
- SQL injection prevention
- XSS protection
- CORS configuration
- Rate limiting
- Security headers

### Security Scanning
```bash
# Scan Docker images
docker scan todo-frontend:latest
docker scan todo-backend:latest

# Run security audit
npm audit
```

## 📈 Performance

### Optimization Features
- Docker multi-stage builds
- Image optimization
- Caching strategies
- Database indexing
- CDN integration
- Load balancing

### Performance Monitoring
- Response time tracking
- Throughput monitoring
- Error rate tracking
- Resource utilization

## 🚨 Troubleshooting

### Common Issues

1. **Database connection issues:**
   ```bash
   # Check database status
   docker-compose exec db psql -U user -d myapp
   
   # Check logs
   docker-compose logs db
   ```

2. **Frontend build issues:**
   ```bash
   # Clear node modules
   docker-compose exec frontend rm -rf node_modules
   docker-compose exec frontend npm install
   ```

3. **Backend API issues:**
   ```bash
   # Check API health
   curl http://localhost:8000/health
   
   # Check logs
   docker-compose logs backend
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

### Docker Concepts
- ✅ Multi-stage builds
- ✅ Multi-service applications
- ✅ Volume management
- ✅ Network configuration
- ✅ Health checks
- ✅ Environment management

### Kubernetes Concepts
- ✅ Deployments and Services
- ✅ ConfigMaps and Secrets
- ✅ Ingress configuration
- ✅ Persistent volumes
- ✅ Monitoring and logging
- ✅ Security policies

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