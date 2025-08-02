# Lab 11: Multi-Stage Builds

## 🎯 Learning Objectives
- Master multi-stage Docker builds
- Optimize image size and build time
- Use different base images for build and runtime
- Implement security best practices in builds
- Create production-ready optimized images

## ⏱️ Estimated Time: 70 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of Dockerfile syntax
- Knowledge of programming languages (Python, Node.js, Go)

## 🚀 Lab Exercises

### Exercise 1: Basic Multi-Stage Build

1. **Create a simple Node.js application:**
   ```bash
   cat > app.js << EOF
   const express = require('express');
   const app = express();
   const port = process.env.PORT || 3000;
   
   app.get('/', (req, res) => {
     res.json({ message: 'Hello from Multi-Stage Build!', timestamp: new Date() });
   });
   
   app.get('/health', (req, res) => {
     res.json({ status: 'healthy', uptime: process.uptime() });
   });
   
   app.listen(port, () => {
     console.log(\`Server running on port \${port}\`);
   });
   EOF
   ```

2. **Create package.json:**
   ```bash
   cat > package.json << EOF
   {
     "name": "multi-stage-app",
     "version": "1.0.0",
     "main": "app.js",
     "scripts": {
       "start": "node app.js",
       "dev": "nodemon app.js"
     },
     "dependencies": {
       "express": "^4.17.1"
     },
     "devDependencies": {
       "nodemon": "^2.0.12"
     }
   }
   EOF
   ```

3. **Create a basic multi-stage Dockerfile:**
   ```bash
   cat > Dockerfile.basic-multistage << EOF
   # Build stage
   FROM node:16-alpine AS builder
   
   WORKDIR /app
   
   # Copy package files
   COPY package*.json ./
   
   # Install dependencies
   RUN npm ci --only=production
   
   # Copy source code
   COPY . .
   
   # Production stage
   FROM node:16-alpine AS production
   
   # Create non-root user
   RUN addgroup -g 1001 -S nodejs && \\
       adduser -S nodejs -u 1001
   
   WORKDIR /app
   
   # Copy dependencies from builder
   COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
   
   # Copy application code
   COPY --from=builder --chown=nodejs:nodejs /app/app.js ./
   
   # Switch to non-root user
   USER nodejs
   
   EXPOSE 3000
   
   CMD ["npm", "start"]
   EOF
   ```

4. **Build and run the multi-stage image:**
   ```bash
   docker build -f Dockerfile.basic-multistage -t multistage-basic .
   docker run -d --name multistage-demo -p 3000:3000 multistage-basic
   ```

5. **Test the application:**
   ```bash
   curl http://localhost:3000/
   curl http://localhost:3000/health
   ```

### Exercise 2: Advanced Multi-Stage with Testing

1. **Create a Python application with tests:**
   ```bash
   cat > main.py << EOF
   from flask import Flask, jsonify
   import os
   import time
   
   app = Flask(__name__)
   
   @app.route('/')
   def hello():
       return jsonify({
           'message': 'Hello from Python Multi-Stage!',
           'timestamp': time.time(),
           'environment': os.getenv('ENVIRONMENT', 'development')
       })
   
   @app.route('/health')
   def health():
       return jsonify({'status': 'healthy'})
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

2. **Create requirements files:**
   ```bash
   cat > requirements.txt << EOF
   flask==2.0.1
   gunicorn==20.1.0
   EOF
   
   cat > requirements-dev.txt << EOF
   -r requirements.txt
   pytest==6.2.5
   pytest-cov==2.12.1
   flake8==3.9.2
   EOF
   ```

3. **Create test file:**
   ```bash
   cat > test_main.py << EOF
   import pytest
   from main import app
   
   @pytest.fixture
   def client():
       app.config['TESTING'] = True
       with app.test_client() as client:
           yield client
   
   def test_hello_endpoint(client):
       response = client.get('/')
       assert response.status_code == 200
       data = response.get_json()
       assert 'message' in data
       assert 'timestamp' in data
   
   def test_health_endpoint(client):
       response = client.get('/health')
       assert response.status_code == 200
       data = response.get_json()
       assert data['status'] == 'healthy'
   EOF
   ```

4. **Create advanced multi-stage Dockerfile:**
   ```bash
   cat > Dockerfile.advanced-multistage << EOF
   # Base stage for common dependencies
   FROM python:3.9-slim AS base
   
   # Install system dependencies
   RUN apt-get update && apt-get install -y \\
       gcc \\
       && rm -rf /var/lib/apt/lists/*
   
   # Set environment variables
   ENV PYTHONDONTWRITEBYTECODE=1 \\
       PYTHONUNBUFFERED=1 \\
       PIP_NO_CACHE_DIR=1
   
   # Create non-root user
   RUN groupadd -r appuser && useradd -r -g appuser appuser
   
   # Test stage
   FROM base AS test
   
   WORKDIR /app
   
   # Copy requirements for testing
   COPY requirements*.txt ./
   
   # Install all dependencies (including dev)
   RUN pip install -r requirements-dev.txt
   
   # Copy source code
   COPY . .
   
   # Run tests
   RUN pytest --cov=main --cov-report=term-missing
   
   # Lint check
   RUN flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
   
   # Build stage
   FROM base AS builder
   
   WORKDIR /app
   
   # Copy requirements
   COPY requirements.txt .
   
   # Install only production dependencies
   RUN pip install -r requirements.txt
   
   # Production stage
   FROM python:3.9-slim AS production
   
   # Install runtime dependencies only
   RUN apt-get update && apt-get install -y \\
       curl \\
       && rm -rf /var/lib/apt/lists/*
   
   # Create non-root user
   RUN groupadd -r appuser && useradd -r -g appuser appuser
   
   WORKDIR /app
   
   # Copy Python packages from builder
   COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
   COPY --from=builder /usr/local/bin /usr/local/bin
   
   # Copy application code
   COPY --chown=appuser:appuser main.py .
   
   # Switch to non-root user
   USER appuser
   
   EXPOSE 5000
   
   # Health check
   HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \\
     CMD curl -f http://localhost:5000/health || exit 1
   
   CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "main:app"]
   EOF
   ```

5. **Build and test the advanced multi-stage image:**
   ```bash
   docker build -f Dockerfile.advanced-multistage -t multistage-advanced .
   docker run -d --name multistage-advanced-demo -p 5000:5000 multistage-advanced
   ```

6. **Test the application:**
   ```bash
   curl http://localhost:5000/
   curl http://localhost:5000/health
   ```

### Exercise 3: Go Multi-Stage Build

1. **Create a Go application:**
   ```bash
   cat > main.go << EOF
   package main
   
   import (
       "encoding/json"
       "log"
       "net/http"
       "os"
       "time"
   )
   
   type Response struct {
       Message   string  \`json:"message"\`
       Timestamp float64 \`json:"timestamp"\`
       Version   string  \`json:"version"\`
   }
   
   type HealthResponse struct {
       Status string \`json:"status"\`
       Uptime string \`json:"uptime"\`
   }
   
   var startTime = time.Now()
   
   func main() {
       http.HandleFunc("/", handleRoot)
       http.HandleFunc("/health", handleHealth)
       
       port := os.Getenv("PORT")
       if port == "" {
           port = "8080"
       }
       
       log.Printf("Server starting on port %s", port)
       log.Fatal(http.ListenAndServe(":"+port, nil))
   }
   
   func handleRoot(w http.ResponseWriter, r *http.Request) {
       response := Response{
           Message:   "Hello from Go Multi-Stage Build!",
           Timestamp: float64(time.Now().Unix()),
           Version:   "1.0.0",
       }
       
       w.Header().Set("Content-Type", "application/json")
       json.NewEncoder(w).Encode(response)
   }
   
   func handleHealth(w http.ResponseWriter, r *http.Request) {
       response := HealthResponse{
           Status: "healthy",
           Uptime: time.Since(startTime).String(),
       }
       
       w.Header().Set("Content-Type", "application/json")
       json.NewEncoder(w).Encode(response)
   }
   EOF
   ```

2. **Create Go module file:**
   ```bash
   cat > go.mod << EOF
   module multistage-go
   
   go 1.19
   EOF
   ```

3. **Create Go multi-stage Dockerfile:**
   ```bash
   cat > Dockerfile.go-multistage << EOF
   # Build stage
   FROM golang:1.19-alpine AS builder
   
   # Install git and ca-certificates
   RUN apk add --no-cache git ca-certificates
   
   # Set working directory
   WORKDIR /app
   
   # Copy go mod files
   COPY go.mod go.sum* ./
   
   # Download dependencies
   RUN go mod download
   
   # Copy source code
   COPY . .
   
   # Build the application
   RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
   
   # Production stage
   FROM scratch
   
   # Copy ca-certificates from builder
   COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
   
   # Copy binary from builder
   COPY --from=builder /app/main .
   
   # Expose port
   EXPOSE 8080
   
   # Run the binary
   CMD ["./main"]
   EOF
   ```

4. **Build and run the Go multi-stage image:**
   ```bash
   docker build -f Dockerfile.go-multistage -t multistage-go .
   docker run -d --name multistage-go-demo -p 8080:8080 multistage-go
   ```

5. **Test the Go application:**
   ```bash
   curl http://localhost:8080/
   curl http://localhost:8080/health
   ```

### Exercise 4: Frontend Multi-Stage Build

1. **Create a React application structure:**
   ```bash
   mkdir -p frontend/src
   ```

2. **Create package.json for React app:**
   ```bash
   cat > frontend/package.json << EOF
   {
     "name": "react-multistage",
     "version": "1.0.0",
     "private": true,
     "dependencies": {
       "react": "^17.0.2",
       "react-dom": "^17.0.2",
       "react-scripts": "4.0.3"
     },
     "scripts": {
       "start": "react-scripts start",
       "build": "react-scripts build",
       "test": "react-scripts test",
       "eject": "react-scripts eject"
     },
     "browserslist": {
       "production": [
         ">0.2%",
         "not dead",
         "not op_mini all"
       ],
       "development": [
         "last 1 chrome version",
         "last 1 firefox version",
         "last 1 safari version"
       ]
     }
   }
   EOF
   ```

3. **Create React app files:**
   ```bash
   cat > frontend/public/index.html << EOF
   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="utf-8" />
       <meta name="viewport" content="width=device-width, initial-scale=1" />
       <title>React Multi-Stage App</title>
     </head>
     <body>
       <div id="root"></div>
     </body>
   </html>
   EOF
   
   cat > frontend/src/App.js << EOF
   import React from 'react';
   import './App.css';
   
   function App() {
     return (
       <div className="App">
         <header className="App-header">
           <h1>React Multi-Stage Build</h1>
           <p>Built with Docker multi-stage builds!</p>
           <p>Version: 1.0.0</p>
         </header>
       </div>
     );
   }
   
   export default App;
   EOF
   
   cat > frontend/src/App.css << EOF
   .App {
     text-align: center;
   }
   
   .App-header {
     background-color: #282c34;
     padding: 20px;
     color: white;
     min-height: 100vh;
     display: flex;
     flex-direction: column;
     align-items: center;
     justify-content: center;
   }
   
   .App-header h1 {
     font-size: 3rem;
     margin-bottom: 1rem;
   }
   
   .App-header p {
     font-size: 1.2rem;
     margin: 0.5rem 0;
   }
   EOF
   
   cat > frontend/src/index.js << EOF
   import React from 'react';
   import ReactDOM from 'react-dom';
   import './App.css';
   import App from './App';
   
   ReactDOM.render(
     <React.StrictMode>
       <App />
     </React.StrictMode>,
     document.getElementById('root')
   );
   EOF
   ```

4. **Create React multi-stage Dockerfile:**
   ```bash
   cat > Dockerfile.react-multistage << EOF
   # Build stage
   FROM node:16-alpine AS builder
   
   WORKDIR /app
   
   # Copy package files
   COPY frontend/package*.json ./
   
   # Install dependencies
   RUN npm ci --only=production
   
   # Copy source code
   COPY frontend/ .
   
   # Build the application
   RUN npm run build
   
   # Production stage
   FROM nginx:alpine
   
   # Copy built files from builder
   COPY --from=builder /app/build /usr/share/nginx/html
   
   # Copy custom nginx config
   COPY nginx-react.conf /etc/nginx/conf.d/default.conf
   
   # Expose port
   EXPOSE 80
   
   # Start nginx
   CMD ["nginx", "-g", "daemon off;"]
   EOF
   ```

5. **Create nginx configuration for React:**
   ```bash
   cat > nginx-react.conf << EOF
   server {
       listen 80;
       server_name localhost;
       
       root /usr/share/nginx/html;
       index index.html index.htm;
       
       # Handle React Router
       location / {
           try_files \$uri \$uri/ /index.html;
       }
       
       # Cache static assets
       location ~* \\.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
           expires 1y;
           add_header Cache-Control "public, immutable";
       }
   }
   EOF
   ```

6. **Build and run the React multi-stage image:**
   ```bash
   docker build -f Dockerfile.react-multistage -t multistage-react .
   docker run -d --name multistage-react-demo -p 3001:80 multistage-react
   ```

7. **Test the React application:**
   ```bash
   curl http://localhost:3001/
   ```

### Exercise 5: Security-Focused Multi-Stage Build

1. **Create a security-focused Dockerfile:**
   ```bash
   cat > Dockerfile.security-multistage << EOF
   # Build stage with all tools
   FROM python:3.9-slim AS builder
   
   # Install build dependencies
   RUN apt-get update && apt-get install -y \\
       gcc \\
       g++ \\
       libffi-dev \\
       && rm -rf /var/lib/apt/lists/*
   
   WORKDIR /app
   
   # Copy requirements
   COPY requirements.txt .
   
   # Install dependencies in virtual environment
   RUN python -m venv /opt/venv
   ENV PATH="/opt/venv/bin:\$PATH"
   RUN pip install --no-cache-dir -r requirements.txt
   
   # Security scan stage
   FROM builder AS security-scan
   
   # Install security tools
   RUN pip install bandit safety
   
   # Copy application code
   COPY main.py .
   
   # Run security scans
   RUN bandit -r . -f json -o bandit-report.json || true
   RUN safety check --json --output safety-report.json || true
   
   # Production stage with minimal attack surface
   FROM python:3.9-slim AS production
   
   # Install only runtime dependencies
   RUN apt-get update && apt-get install -y \\
       curl \\
       && rm -rf /var/lib/apt/lists/*
   
   # Create non-root user
   RUN groupadd -r appuser && useradd -r -g appuser appuser
   
   WORKDIR /app
   
   # Copy virtual environment from builder
   COPY --from=builder /opt/venv /opt/venv
   ENV PATH="/opt/venv/bin:\$PATH"
   
   # Copy application code
   COPY --chown=appuser:appuser main.py .
   
   # Switch to non-root user
   USER appuser
   
   EXPOSE 5000
   
   # Health check
   HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \\
     CMD curl -f http://localhost:5000/health || exit 1
   
   CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "main:app"]
   EOF
   ```

2. **Build the security-focused image:**
   ```bash
   docker build -f Dockerfile.security-multistage -t multistage-security .
   ```

3. **Check security scan results:**
   ```bash
   # Extract security reports from build
   docker create --name temp-container multistage-security
   docker cp temp-container:/app/bandit-report.json ./bandit-report.json
   docker cp temp-container:/app/safety-report.json ./safety-report.json
   docker rm temp-container
   
   # View reports
   cat bandit-report.json
   cat safety-report.json
   ```

## 🧹 Cleanup

1. **Stop and remove containers:**
   ```bash
   docker stop multistage-demo multistage-advanced-demo multistage-go-demo multistage-react-demo
   docker rm multistage-demo multistage-advanced-demo multistage-go-demo multistage-react-demo
   ```

2. **Remove images:**
   ```bash
   docker rmi multistage-basic multistage-advanced multistage-go multistage-react multistage-security
   ```

3. **Clean up files:**
   ```bash
   rm -f Dockerfile.* app.js package.json main.py requirements*.txt test_main.py \
        main.go go.mod go.sum nginx-react.conf bandit-report.json safety-report.json
   rm -rf frontend/
   ```

## 📚 Key Takeaways

- **Multi-Stage Benefits**: Significantly reduce final image size
- **Security**: Use different stages for build and runtime
- **Testing**: Include testing in build stages
- **Optimization**: Use appropriate base images for each stage
- **Best Practices**: Copy only necessary files to production stage

## 🔍 Troubleshooting

### Common Issues:
1. **Build failures**: Check base image compatibility
2. **Large image sizes**: Ensure proper file copying between stages
3. **Security vulnerabilities**: Use security scanning in build stages
4. **Runtime errors**: Verify all dependencies are copied to production stage

### Debug Commands:
```bash
# Check image layers
docker history <image>

# Compare image sizes
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Inspect build stages
docker build --target <stage> -t <image> .

# Check security scan results
docker run --rm -v $(pwd):/app <image> cat /app/security-report.json
```

## 🎯 Next Steps

- Implement multi-stage builds in CI/CD pipelines
- Learn about BuildKit for advanced build features
- Explore security scanning tools integration
- Study Kubernetes multi-stage build patterns
- Practice with complex application architectures 