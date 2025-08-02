# Lab 06: Docker Compose Basics

## 🎯 Learning Objectives
- Understand Docker Compose concepts and syntax
- Create multi-service applications
- Manage service dependencies
- Configure networks and volumes
- Practice service orchestration

## ⏱️ Estimated Time: 60 minutes

## 📋 Prerequisites
- Docker installed and running
- Docker Compose installed
- Basic container operations (Labs 1-5)
- Understanding of networking and volumes

## 🚀 Lab Exercises

### Exercise 1: Your First Docker Compose Application

1. **Create a simple web application:**
   ```bash
   # Create project directory
   mkdir docker-compose-demo
   cd docker-compose-demo
   
   # Create HTML file
   cat << EOF > index.html
   <!DOCTYPE html>
   <html>
   <head><title>Docker Compose Demo</title></head>
   <body>
       <h1>Hello from Docker Compose!</h1>
       <p>This is a multi-service application.</p>
   </body>
   </html>
   EOF
   ```

2. **Create docker-compose.yml:**
   ```bash
   # Create Docker Compose file
   cat << EOF > docker-compose.yml
   version: '3.8'
   
   services:
     web:
       image: nginx:alpine
       ports:
         - "8080:80"
       volumes:
         - ./index.html:/usr/share/nginx/html/index.html
       depends_on:
         - db
       networks:
         - app-network
   
     db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: myapp
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - postgres_data:/var/lib/postgresql/data
       networks:
         - app-network
   
     cache:
       image: redis:alpine
       networks:
         - app-network
   
   volumes:
     postgres_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   ```

3. **Run the application:**
   ```bash
   # Start all services
   docker-compose up -d
   
   # Check service status
   docker-compose ps
   
   # View logs
   docker-compose logs
   
   # Access the application
   curl http://localhost:8080
   ```

### Exercise 2: Service Management

1. **Manage individual services:**
   ```bash
   # Start specific service
   docker-compose up -d web
   
   # Stop specific service
   docker-compose stop db
   
   # Restart service
   docker-compose restart cache
   
   # Scale service
   docker-compose up -d --scale web=3
   ```

2. **Service operations:**
   ```bash
   # View running services
   docker-compose ps
   
   # View service logs
   docker-compose logs web
   docker-compose logs db
   
   # Execute commands in services
   docker-compose exec web sh
   docker-compose exec db psql -U user -d myapp
   ```

### Exercise 3: Environment Configuration

1. **Create environment file:**
   ```bash
   # Create .env file
   cat << EOF > .env
   POSTGRES_DB=myapp
   POSTGRES_USER=user
   POSTGRES_PASSWORD=password
   REDIS_PASSWORD=redis123
   NODE_ENV=development
   EOF
   ```

2. **Update docker-compose.yml with environment:**
   ```bash
   # Update docker-compose.yml
   cat << EOF > docker-compose.yml
   version: '3.8'
   
   services:
     web:
       image: nginx:alpine
       ports:
         - "8080:80"
       volumes:
         - ./index.html:/usr/share/nginx/html/index.html
       environment:
         - NODE_ENV=\${NODE_ENV}
       depends_on:
         - db
       networks:
         - app-network
   
     db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: \${POSTGRES_DB}
         POSTGRES_USER: \${POSTGRES_USER}
         POSTGRES_PASSWORD: \${POSTGRES_PASSWORD}
       volumes:
         - postgres_data:/var/lib/postgresql/data
       networks:
         - app-network
   
     cache:
       image: redis:alpine
       command: redis-server --requirepass \${REDIS_PASSWORD}
       networks:
         - app-network
   
   volumes:
     postgres_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   ```

3. **Test environment variables:**
   ```bash
   # Restart with new environment
   docker-compose down
   docker-compose up -d
   
   # Check environment variables
   docker-compose exec db env | grep POSTGRES
   docker-compose exec cache redis-cli -a redis123 ping
   ```

### Exercise 4: Multi-Service Communication

1. **Create a Node.js application:**
   ```bash
   # Create package.json
   cat << EOF > package.json
   {
     "name": "compose-demo",
     "version": "1.0.0",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.18.2",
       "pg": "^8.11.0",
       "redis": "^4.6.0"
     }
   }
   EOF
   
   # Create app.js
   cat << EOF > app.js
   const express = require('express');
   const { Client } = require('pg');
   const redis = require('redis');
   
   const app = express();
   const port = 3000;
   
   // PostgreSQL connection
   const pgClient = new Client({
     host: 'db',
     port: 5432,
     database: process.env.POSTGRES_DB,
     user: process.env.POSTGRES_USER,
     password: process.env.POSTGRES_PASSWORD,
   });
   
   // Redis connection
   const redisClient = redis.createClient({
     host: 'cache',
     port: 6379,
     password: process.env.REDIS_PASSWORD
   });
   
   app.get('/', async (req, res) => {
     try {
       // Test PostgreSQL
       await pgClient.query('SELECT NOW()');
       
       // Test Redis
       await redisClient.set('test', 'Hello from Redis!');
       const redisValue = await redisClient.get('test');
       
       res.json({
         message: 'Hello from Docker Compose!',
         postgres: 'Connected',
         redis: redisValue,
         timestamp: new Date().toISOString()
       });
     } catch (error) {
       res.status(500).json({ error: error.message });
     }
   });
   
   // Connect to databases
   async function connect() {
     try {
       await pgClient.connect();
       await redisClient.connect();
       console.log('Connected to databases');
     } catch (error) {
       console.error('Database connection error:', error);
     }
   }
   
   connect();
   
   app.listen(port, () => {
     console.log(\`App listening at http://localhost:\${port}\`);
   });
   EOF
   ```

2. **Create Dockerfile for Node.js app:**
   ```bash
   # Create Dockerfile
   cat << EOF > Dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["npm", "start"]
   EOF
   ```

3. **Update docker-compose.yml:**
   ```bash
   # Update docker-compose.yml
   cat << EOF > docker-compose.yml
   version: '3.8'
   
   services:
     web:
       build: .
       ports:
         - "3000:3000"
       environment:
         - NODE_ENV=\${NODE_ENV}
         - POSTGRES_DB=\${POSTGRES_DB}
         - POSTGRES_USER=\${POSTGRES_USER}
         - POSTGRES_PASSWORD=\${POSTGRES_PASSWORD}
         - REDIS_PASSWORD=\${REDIS_PASSWORD}
       depends_on:
         - db
         - cache
       networks:
         - app-network
   
     db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: \${POSTGRES_DB}
         POSTGRES_USER: \${POSTGRES_USER}
         POSTGRES_PASSWORD: \${POSTGRES_PASSWORD}
       volumes:
         - postgres_data:/var/lib/postgresql/data
       networks:
         - app-network
   
     cache:
       image: redis:alpine
       command: redis-server --requirepass \${REDIS_PASSWORD}
       networks:
         - app-network
   
   volumes:
     postgres_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   ```

4. **Test multi-service communication:**
   ```bash
   # Build and start services
   docker-compose up -d --build
   
   # Test the application
   curl http://localhost:3000
   
   # Check service logs
   docker-compose logs web
   ```

### Exercise 5: Advanced Compose Features

1. **Create development and production configurations:**
   ```bash
   # Create docker-compose.dev.yml
   cat << EOF > docker-compose.dev.yml
   version: '3.8'
   
   services:
     web:
       build: .
       ports:
         - "3000:3000"
       volumes:
         - .:/app
         - /app/node_modules
       environment:
         - NODE_ENV=development
       command: npm run dev
       depends_on:
         - db
         - cache
       networks:
         - app-network
   
     db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: myapp_dev
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - postgres_dev_data:/var/lib/postgresql/data
       networks:
         - app-network
   
     cache:
       image: redis:alpine
       networks:
         - app-network
   
   volumes:
     postgres_dev_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   
   # Create docker-compose.prod.yml
   cat << EOF > docker-compose.prod.yml
   version: '3.8'
   
   services:
     web:
       image: compose-demo:latest
       ports:
         - "80:3000"
       environment:
         - NODE_ENV=production
       depends_on:
         - db
         - cache
       networks:
         - app-network
       restart: unless-stopped
   
     db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: myapp_prod
         POSTGRES_USER: user
         POSTGRES_PASSWORD: \${DB_PASSWORD}
       volumes:
         - postgres_prod_data:/var/lib/postgresql/data
       networks:
         - app-network
       restart: unless-stopped
   
     cache:
       image: redis:alpine
       command: redis-server --requirepass \${REDIS_PASSWORD}
       networks:
         - app-network
       restart: unless-stopped
   
   volumes:
     postgres_prod_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   ```

2. **Use different configurations:**
   ```bash
   # Run development environment
   docker-compose -f docker-compose.dev.yml up -d
   
   # Stop development
   docker-compose -f docker-compose.dev.yml down
   
   # Run production environment
   docker-compose -f docker-compose.prod.yml up -d
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Microservices Application

1. **Create a microservices architecture:**
   ```bash
   # Create microservices docker-compose.yml
   cat << EOF > docker-compose.microservices.yml
   version: '3.8'
   
   services:
     api-gateway:
       image: nginx:alpine
       ports:
         - "80:80"
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf
       depends_on:
         - user-service
         - product-service
       networks:
         - gateway-network
         - services-network
   
     user-service:
       build: ./user-service
       environment:
         - DB_HOST=user-db
         - REDIS_HOST=redis
       depends_on:
         - user-db
         - redis
       networks:
         - services-network
   
     product-service:
       build: ./product-service
       environment:
         - DB_HOST=product-db
         - REDIS_HOST=redis
       depends_on:
         - product-db
         - redis
       networks:
         - services-network
   
     user-db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: users
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - user_data:/var/lib/postgresql/data
       networks:
         - services-network
   
     product-db:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: products
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - product_data:/var/lib/postgresql/data
       networks:
         - services-network
   
     redis:
       image: redis:alpine
       networks:
         - services-network
   
   volumes:
     user_data:
     product_data:
   
   networks:
     gateway-network:
       driver: bridge
     services-network:
       driver: bridge
   EOF
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you create a Docker Compose file?
- [ ] Do you understand service dependencies?
- [ ] Can you manage multiple services?
- [ ] Can you configure environment variables?
- [ ] Do you understand networks and volumes in Compose?
- [ ] Can you create development and production configurations?

### Skills Demonstrated:
- ✅ Docker Compose syntax and concepts
- ✅ Multi-service application creation
- ✅ Service dependency management
- ✅ Environment configuration
- ✅ Network and volume configuration
- ✅ Development and production setups

## 🔍 Troubleshooting

### Common Issues:

1. **Service won't start:**
   ```bash
   # Check service logs
   docker-compose logs <service-name>
   
   # Check service status
   docker-compose ps
   
   # Check for port conflicts
   docker-compose ps
   netstat -tulpn | grep :8080
   ```

2. **Service communication issues:**
   ```bash
   # Check network connectivity
   docker-compose exec <service-name> ping <other-service>
   
   # Check network configuration
   docker network ls
   docker network inspect <network-name>
   ```

3. **Volume mounting issues:**
   ```bash
   # Check volume configuration
   docker volume ls
   docker-compose exec <service-name> ls -la /path/to/mount
   
   # Check file permissions
   docker-compose exec <service-name> ls -la /path/to/mount
   ```

## 📚 Additional Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Docker Compose Best Practices](https://docs.docker.com/compose/production/)

## 🎉 Lab Completion

Congratulations! You've completed the Docker Compose Basics lab. You now understand:
- Docker Compose concepts and syntax
- Multi-service application creation
- Service dependency management
- Environment configuration
- Network and volume configuration

**Next Lab**: [Lab 07: Multi-Service Applications](../docker/lab-07-multi-service-applications.md)

---

**Happy Learning! 🐳** 