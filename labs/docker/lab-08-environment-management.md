# Lab 08: Environment Management

## 🎯 Learning Objectives
- Manage environment variables in containers
- Use configuration files for application settings
- Implement secrets management
- Understand best practices for configuration

## ⏱️ Estimated Time: 50 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic knowledge of environment variables
- Understanding of Docker run commands

## 🚀 Lab Exercises

### Exercise 1: Environment Variables

1. **Run a container with environment variables:**
   ```bash
   docker run -e DB_HOST=localhost -e DB_PORT=5432 -e APP_ENV=development nginx:alpine
   ```

2. **Create a container with multiple environment variables:**
   ```bash
   docker run -d \
     --name env-demo \
     -e DATABASE_URL=postgresql://user:pass@db:5432/mydb \
     -e REDIS_URL=redis://redis:6379 \
     -e LOG_LEVEL=debug \
     -e API_KEY=your-secret-key \
     nginx:alpine
   ```

3. **Check environment variables in running container:**
   ```bash
   docker exec env-demo env
   ```

### Exercise 2: Environment File

1. **Create an environment file:**
   ```bash
   cat > .env << EOF
   APP_NAME=MyApp
   APP_VERSION=1.0.0
   DATABASE_HOST=db.example.com
   DATABASE_PORT=5432
   DATABASE_NAME=myapp
   DATABASE_USER=appuser
   DATABASE_PASSWORD=secret123
   REDIS_HOST=redis.example.com
   REDIS_PORT=6379
   LOG_LEVEL=info
   EOF
   ```

2. **Run container using environment file:**
   ```bash
   docker run --env-file .env nginx:alpine
   ```

3. **Verify environment variables:**
   ```bash
   docker run --env-file .env nginx:alpine env | grep -E "(APP_|DATABASE_|REDIS_|LOG_)"
   ```

### Exercise 3: Configuration Files

1. **Create a configuration directory:**
   ```bash
   mkdir -p config
   ```

2. **Create application configuration:**
   ```bash
   cat > config/app.conf << EOF
   [database]
   host = db.example.com
   port = 5432
   name = myapp
   user = appuser
   password = secret123

   [redis]
   host = redis.example.com
   port = 6379

   [logging]
   level = info
   format = json
   EOF
   ```

3. **Run container with configuration mount:**
   ```bash
   docker run -d \
     --name config-demo \
     -v $(pwd)/config:/app/config \
     nginx:alpine
   ```

4. **Verify configuration file:**
   ```bash
   docker exec config-demo cat /app/config/app.conf
   ```

### Exercise 4: Secrets Management

1. **Create a secrets directory:**
   ```bash
   mkdir -p secrets
   ```

2. **Create secret files:**
   ```bash
   echo "super-secret-password" > secrets/db_password
   echo "api-key-12345" > secrets/api_key
   echo "jwt-secret-key" > secrets/jwt_secret
   ```

3. **Set proper permissions:**
   ```bash
   chmod 600 secrets/*
   ```

4. **Run container with secrets:**
   ```bash
   docker run -d \
     --name secrets-demo \
     -v $(pwd)/secrets:/run/secrets:ro \
     nginx:alpine
   ```

5. **Access secrets in container:**
   ```bash
   docker exec secrets-demo cat /run/secrets/db_password
   docker exec secrets-demo cat /run/secrets/api_key
   ```

### Exercise 5: Docker Compose with Environment

1. **Create docker-compose.yml with environment variables:**
   ```bash
   cat > docker-compose.yml << EOF
   version: '3.8'
   
   services:
     web:
       image: nginx:alpine
       environment:
         - APP_ENV=development
         - DATABASE_URL=postgresql://user:pass@db:5432/mydb
         - REDIS_URL=redis://redis:6379
       env_file:
         - .env
       volumes:
         - ./config:/app/config:ro
         - ./secrets:/run/secrets:ro
       ports:
         - "8080:80"
   
     db:
       image: postgres:13-alpine
       environment:
         - POSTGRES_DB=mydb
         - POSTGRES_USER=appuser
         - POSTGRES_PASSWORD=secret123
       volumes:
         - postgres_data:/var/lib/postgresql/data
   
     redis:
       image: redis:alpine
       command: redis-server --requirepass secret123
   
   volumes:
     postgres_data:
   EOF
   ```

2. **Start the services:**
   ```bash
   docker-compose up -d
   ```

3. **Check environment variables:**
   ```bash
   docker-compose exec web env | grep -E "(APP_|DATABASE_|REDIS_)"
   ```

### Exercise 6: Environment Variable Best Practices

1. **Create a production environment file:**
   ```bash
   cat > .env.production << EOF
   # Application Configuration
   APP_ENV=production
   APP_DEBUG=false
   APP_LOG_LEVEL=warn
   
   # Database Configuration
   DATABASE_HOST=prod-db.example.com
   DATABASE_PORT=5432
   DATABASE_NAME=myapp_prod
   DATABASE_USER=appuser
   DATABASE_PASSWORD=prod-secret-password
   
   # Redis Configuration
   REDIS_HOST=prod-redis.example.com
   REDIS_PORT=6379
   REDIS_PASSWORD=prod-redis-password
   
   # API Configuration
   API_BASE_URL=https://api.example.com
   API_TIMEOUT=30
   EOF
   ```

2. **Create a development environment file:**
   ```bash
   cat > .env.development << EOF
   # Application Configuration
   APP_ENV=development
   APP_DEBUG=true
   APP_LOG_LEVEL=debug
   
   # Database Configuration
   DATABASE_HOST=localhost
   DATABASE_PORT=5432
   DATABASE_NAME=myapp_dev
   DATABASE_USER=devuser
   DATABASE_PASSWORD=dev-password
   
   # Redis Configuration
   REDIS_HOST=localhost
   REDIS_PORT=6379
   REDIS_PASSWORD=
   
   # API Configuration
   API_BASE_URL=http://localhost:3000
   API_TIMEOUT=10
   EOF
   ```

3. **Use different environment files:**
   ```bash
   # Development
   docker run --env-file .env.development nginx:alpine
   
   # Production
   docker run --env-file .env.production nginx:alpine
   ```

## 🧹 Cleanup

1. **Stop and remove containers:**
   ```bash
   docker stop env-demo config-demo secrets-demo
   docker rm env-demo config-demo secrets-demo
   ```

2. **Stop Docker Compose services:**
   ```bash
   docker-compose down -v
   ```

3. **Clean up files:**
   ```bash
   rm -rf .env* config/ secrets/ docker-compose.yml
   ```

## 📚 Key Takeaways

- **Environment Variables**: Use `-e` flag or `--env-file` for configuration
- **Configuration Files**: Mount configuration files as volumes
- **Secrets Management**: Store sensitive data in separate files with restricted permissions
- **Best Practices**: Use different environment files for different environments
- **Security**: Never commit secrets to version control

## 🔍 Troubleshooting

### Common Issues:
1. **Environment variables not set**: Check file permissions and syntax
2. **Configuration files not found**: Verify volume mount paths
3. **Secrets not accessible**: Check file permissions (should be 600)
4. **Docker Compose environment issues**: Ensure .env file exists and is readable

### Debug Commands:
```bash
# Check environment variables in container
docker exec <container> env

# Verify file mounts
docker exec <container> ls -la /path/to/mount

# Check Docker Compose environment
docker-compose config
```

## 🎯 Next Steps

- Practice with different application types (Node.js, Python, Java)
- Implement environment-specific configurations
- Learn about Docker secrets for production deployments
- Explore configuration management tools (Consul, etcd) 