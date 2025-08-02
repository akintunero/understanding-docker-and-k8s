# Lab 07: Multi-Service Applications

## 🎯 Learning Objectives
- Build complex multi-service applications
- Implement service discovery and communication
- Configure load balancing and scaling
- Practice monitoring and logging
- Understand microservices patterns

## ⏱️ Estimated Time: 75 minutes

## 📋 Prerequisites
- Docker and Docker Compose installed
- Understanding of Docker Compose (Lab 06)
- Basic networking concepts

## 🚀 Lab Exercises

### Exercise 1: E-commerce Application

1. **Create the application structure:**
   ```bash
   mkdir ecommerce-app && cd ecommerce-app
   
   # Create services directory
   mkdir -p services/{frontend,backend,payment,database}
   ```

2. **Create frontend service:**
   ```bash
   # Create frontend Dockerfile
   cat << EOF > services/frontend/Dockerfile
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   EOF
   
   # Create frontend HTML
   cat << EOF > services/frontend/index.html
   <!DOCTYPE html>
   <html>
   <head><title>E-commerce App</title></head>
   <body>
       <h1>E-commerce Application</h1>
       <p>Frontend Service</p>
       <div id="products"></div>
       <script>
           fetch('http://backend:3000/api/products')
               .then(response => response.json())
               .then(data => {
                   document.getElementById('products').innerHTML = 
                       '<h2>Products:</h2><pre>' + JSON.stringify(data, null, 2) + '</pre>';
               });
       </script>
   </body>
   </html>
   EOF
   ```

3. **Create backend service:**
   ```bash
   # Create backend package.json
   cat << EOF > services/backend/package.json
   {
     "name": "backend",
     "version": "1.0.0",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.18.2",
       "cors": "^2.8.5",
       "pg": "^8.11.0"
     }
   }
   EOF
   
   # Create backend app.js
   cat << EOF > services/backend/app.js
   const express = require('express');
   const cors = require('cors');
   const { Client } = require('pg');
   
   const app = express();
   const port = 3000;
   
   app.use(cors());
   app.use(express.json());
   
   const dbClient = new Client({
     host: 'database',
     port: 5432,
     database: 'ecommerce',
     user: 'user',
     password: 'password'
   });
   
   app.get('/api/products', async (req, res) => {
     try {
       const result = await dbClient.query('SELECT * FROM products');
       res.json(result.rows);
     } catch (error) {
       res.status(500).json({ error: error.message });
     }
   });
   
   app.post('/api/orders', async (req, res) => {
     try {
       const { product_id, quantity } = req.body;
       const result = await dbClient.query(
         'INSERT INTO orders (product_id, quantity) VALUES ($1, $2) RETURNING *',
         [product_id, quantity]
       );
       res.json(result.rows[0]);
     } catch (error) {
       res.status(500).json({ error: error.message });
     }
   });
   
   dbClient.connect().then(() => {
     console.log('Connected to database');
     app.listen(port, () => {
       console.log(\`Backend service running on port \${port}\`);
     });
   });
   EOF
   
   # Create backend Dockerfile
   cat << EOF > services/backend/Dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["npm", "start"]
   EOF
   ```

4. **Create docker-compose.yml:**
   ```bash
   cat << EOF > docker-compose.yml
   version: '3.8'
   
   services:
     frontend:
       build: ./services/frontend
       ports:
         - "80:80"
       depends_on:
         - backend
       networks:
         - app-network
   
     backend:
       build: ./services/backend
       ports:
         - "3000:3000"
       depends_on:
         - database
       environment:
         - NODE_ENV=production
       networks:
         - app-network
   
     database:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: ecommerce
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - postgres_data:/var/lib/postgresql/data
         - ./init.sql:/docker-entrypoint-initdb.d/init.sql
       networks:
         - app-network
   
     payment:
       image: nginx:alpine
       ports:
         - "8080:80"
       volumes:
         - ./services/payment:/usr/share/nginx/html
       networks:
         - app-network
   
   volumes:
     postgres_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   ```

5. **Create database initialization:**
   ```bash
   cat << EOF > init.sql
   CREATE TABLE products (
     id SERIAL PRIMARY KEY,
     name VARCHAR(100) NOT NULL,
     price DECIMAL(10,2) NOT NULL,
     description TEXT
   );
   
   CREATE TABLE orders (
     id SERIAL PRIMARY KEY,
     product_id INTEGER REFERENCES products(id),
     quantity INTEGER NOT NULL,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   INSERT INTO products (name, price, description) VALUES
     ('Laptop', 999.99, 'High-performance laptop'),
     ('Phone', 599.99, 'Smartphone'),
     ('Tablet', 299.99, 'Portable tablet');
   EOF
   ```

6. **Run the application:**
   ```bash
   # Build and start all services
   docker-compose up -d --build
   
   # Check service status
   docker-compose ps
   
   # Access the application
   curl http://localhost
   curl http://localhost:3000/api/products
   ```

### Exercise 2: Service Discovery and Communication

1. **Test service communication:**
   ```bash
   # Test frontend to backend communication
   docker-compose exec frontend wget -O- http://backend:3000/api/products
   
   # Test backend to database communication
   docker-compose exec backend node -e "
   const { Client } = require('pg');
   const client = new Client({
     host: 'database',
     port: 5432,
     database: 'ecommerce',
     user: 'user',
     password: 'password'
   });
   client.connect().then(() => {
     client.query('SELECT COUNT(*) FROM products').then(result => {
       console.log('Product count:', result.rows[0].count);
       client.end();
     });
   });
   "
   ```

2. **Add service health checks:**
   ```bash
   # Update docker-compose.yml with health checks
   cat << EOF > docker-compose.yml
   version: '3.8'
   
   services:
     frontend:
       build: ./services/frontend
       ports:
         - "80:80"
       depends_on:
         backend:
           condition: service_healthy
       networks:
         - app-network
       healthcheck:
         test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:80/"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 40s
   
     backend:
       build: ./services/backend
       ports:
         - "3000:3000"
       depends_on:
         database:
           condition: service_healthy
       environment:
         - NODE_ENV=production
       networks:
         - app-network
       healthcheck:
         test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/api/products"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 40s
   
     database:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: ecommerce
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - postgres_data:/var/lib/postgresql/data
         - ./init.sql:/docker-entrypoint-initdb.d/init.sql
       networks:
         - app-network
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U user -d ecommerce"]
         interval: 10s
         timeout: 5s
         retries: 5
   
     payment:
       image: nginx:alpine
       ports:
         - "8080:80"
       volumes:
         - ./services/payment:/usr/share/nginx/html
       networks:
         - app-network
       healthcheck:
         test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:80/"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 40s
   
   volumes:
     postgres_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   ```

### Exercise 3: Load Balancing and Scaling

1. **Add load balancer:**
   ```bash
   # Create nginx load balancer configuration
   cat << EOF > nginx.conf
   events {
       worker_connections 1024;
   }
   
   http {
       upstream backend {
           server backend:3000;
       }
   
       server {
           listen 80;
           
           location / {
               proxy_pass http://frontend;
               proxy_set_header Host \$host;
               proxy_set_header X-Real-IP \$remote_addr;
           }
           
           location /api/ {
               proxy_pass http://backend;
               proxy_set_header Host \$host;
               proxy_set_header X-Real-IP \$remote_addr;
           }
       }
   }
   EOF
   ```

2. **Update docker-compose.yml with load balancer:**
   ```bash
   # Add load balancer service
   cat << EOF > docker-compose.yml
   version: '3.8'
   
   services:
     load-balancer:
       image: nginx:alpine
       ports:
         - "80:80"
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf
       depends_on:
         - frontend
         - backend
       networks:
         - app-network
   
     frontend:
       build: ./services/frontend
       expose:
         - "80"
       depends_on:
         backend:
           condition: service_healthy
       networks:
         - app-network
       healthcheck:
         test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:80/"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 40s
   
     backend:
       build: ./services/backend
       expose:
         - "3000"
       depends_on:
         database:
           condition: service_healthy
       environment:
         - NODE_ENV=production
       networks:
         - app-network
       healthcheck:
         test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/api/products"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 40s
   
     database:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: ecommerce
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - postgres_data:/var/lib/postgresql/data
         - ./init.sql:/docker-entrypoint-initdb.d/init.sql
       networks:
         - app-network
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U user -d ecommerce"]
         interval: 10s
         timeout: 5s
         retries: 5
   
     payment:
       image: nginx:alpine
       ports:
         - "8080:80"
       volumes:
         - ./services/payment:/usr/share/nginx/html
       networks:
         - app-network
       healthcheck:
         test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:80/"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 40s
   
   volumes:
     postgres_data:
   
   networks:
     app-network:
       driver: bridge
   EOF
   ```

3. **Scale services:**
   ```bash
   # Scale backend service
   docker-compose up -d --scale backend=3
   
   # Check running instances
   docker-compose ps
   
   # Test load balancing
   for i in {1..10}; do
     curl http://localhost/api/products
     echo "Request $i"
   done
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Microservices with Message Queue

1. **Add Redis message queue:**
   ```bash
   # Create order service
   cat << EOF > services/order/package.json
   {
     "name": "order-service",
     "version": "1.0.0",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.18.2",
       "redis": "^4.6.0",
       "pg": "^8.11.0"
     }
   }
   EOF
   
   # Create order service app.js
   cat << EOF > services/order/app.js
   const express = require('express');
   const redis = require('redis');
   const { Client } = require('pg');
   
   const app = express();
   const port = 3001;
   
   app.use(express.json());
   
   const redisClient = redis.createClient({
     host: 'redis',
     port: 6379
   });
   
   const dbClient = new Client({
     host: 'database',
     port: 5432,
     database: 'ecommerce',
     user: 'user',
     password: 'password'
   });
   
   // Process orders from queue
   async function processOrders() {
     while (true) {
       try {
         const order = await redisClient.blPop('orders', 1);
         if (order) {
           console.log('Processing order:', order);
           // Process order logic here
         }
       } catch (error) {
         console.error('Error processing order:', error);
       }
     }
   }
   
   app.post('/orders', async (req, res) => {
     try {
       const order = req.body;
       await redisClient.lPush('orders', JSON.stringify(order));
       res.json({ message: 'Order queued successfully' });
     } catch (error) {
       res.status(500).json({ error: error.message });
     }
   });
   
   redisClient.connect().then(() => {
     console.log('Connected to Redis');
     processOrders();
   });
   
   dbClient.connect().then(() => {
     console.log('Connected to database');
     app.listen(port, () => {
       console.log(\`Order service running on port \${port}\`);
     });
   });
   EOF
   ```

2. **Update docker-compose.yml:**
   ```bash
   # Add Redis and order service
   cat << EOF > docker-compose.yml
   version: '3.8'
   
   services:
     load-balancer:
       image: nginx:alpine
       ports:
         - "80:80"
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf
       depends_on:
         - frontend
         - backend
       networks:
         - app-network
   
     frontend:
       build: ./services/frontend
       expose:
         - "80"
       depends_on:
         backend:
           condition: service_healthy
       networks:
         - app-network
   
     backend:
       build: ./services/backend
       expose:
         - "3000"
       depends_on:
         database:
           condition: service_healthy
       environment:
         - NODE_ENV=production
       networks:
         - app-network
   
     order-service:
       build: ./services/order
       expose:
         - "3001"
       depends_on:
         - redis
         - database
       networks:
         - app-network
   
     database:
       image: postgres:13-alpine
       environment:
         POSTGRES_DB: ecommerce
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - postgres_data:/var/lib/postgresql/data
         - ./init.sql:/docker-entrypoint-initdb.d/init.sql
       networks:
         - app-network
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U user -d ecommerce"]
         interval: 10s
         timeout: 5s
         retries: 5
   
     redis:
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

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you create multi-service applications?
- [ ] Do you understand service discovery?
- [ ] Can you implement load balancing?
- [ ] Can you configure health checks?
- [ ] Do you understand microservices patterns?
- [ ] Can you implement message queues?

### Skills Demonstrated:
- ✅ Multi-service application design
- ✅ Service discovery and communication
- ✅ Load balancing implementation
- ✅ Health check configuration
- ✅ Microservices patterns
- ✅ Message queue integration

## 🔍 Troubleshooting

### Common Issues:

1. **Service communication failures:**
   ```bash
   # Check network connectivity
   docker-compose exec frontend ping backend
   
   # Check service logs
   docker-compose logs backend
   
   # Check DNS resolution
   docker-compose exec frontend nslookup backend
   ```

2. **Database connection issues:**
   ```bash
   # Check database status
   docker-compose exec database pg_isready -U user -d ecommerce
   
   # Check database logs
   docker-compose logs database
   
   # Test database connection
   docker-compose exec backend node -e "
   const { Client } = require('pg');
   const client = new Client({
     host: 'database',
     port: 5432,
     database: 'ecommerce',
     user: 'user',
     password: 'password'
   });
   client.connect().then(() => console.log('Connected')).catch(console.error);
   "
   ```

## 📚 Additional Resources

- [Docker Compose Multi-Service](https://docs.docker.com/compose/networking/)
- [Microservices Patterns](https://microservices.io/patterns/)
- [Service Discovery](https://docs.docker.com/compose/networking/#links)

## 🎉 Lab Completion

Congratulations! You've completed the Multi-Service Applications lab. You now understand:
- Multi-service application architecture
- Service discovery and communication
- Load balancing and scaling
- Health checks and monitoring
- Microservices patterns

**Next Lab**: [Lab 08: Environment Management](../docker/lab-08-environment-management.md)

---

**Happy Learning! 🐳** 