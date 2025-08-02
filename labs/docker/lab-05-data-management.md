# Lab 05: Data Management with Volumes

## 🎯 Learning Objectives
- Understand Docker volume types and use cases
- Create and manage named volumes
- Implement bind mounts for development
- Configure data persistence strategies
- Practice volume backup and restore

## ⏱️ Estimated Time: 45 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic container operations (Labs 1-4)
- Understanding of file systems

## 🚀 Lab Exercises

### Exercise 1: Understanding Volume Types

1. **List existing volumes:**
   ```bash
   # List all volumes
   docker volume ls
   
   # Get detailed volume information
   docker volume inspect $(docker volume ls -q | head -1)
   ```

2. **Create different volume types:**
   ```bash
   # Create named volume
   docker volume create my-named-volume
   
   # Create volume with specific driver
   docker volume create --driver local my-local-volume
   
   # Create volume with labels
   docker volume create --label env=production --label app=web my-labeled-volume
   ```

### Exercise 2: Named Volumes

1. **Create containers with named volumes:**
   ```bash
   # Create database with named volume
   docker run -d --name postgres-db \
     -e POSTGRES_PASSWORD=password \
     -e POSTGRES_DB=myapp \
     -v postgres-data:/var/lib/postgresql/data \
     postgres:13-alpine
   
   # Create web app with named volume
   docker run -d --name web-app \
     -v web-logs:/app/logs \
     -v web-config:/app/config \
     nginx:alpine
   ```

2. **Inspect volume usage:**
   ```bash
   # Check volume details
   docker volume inspect postgres-data
   docker volume inspect web-logs
   
   # List volumes with containers
   docker volume ls --format "table {{.Name}}\t{{.Driver}}\t{{.Mountpoint}}"
   ```

3. **Test data persistence:**
   ```bash
   # Create data in container
   docker exec postgres-db psql -U postgres -d myapp -c "CREATE TABLE test (id SERIAL, name TEXT);"
   docker exec postgres-db psql -U postgres -d myapp -c "INSERT INTO test (name) VALUES ('test data');"
   
   # Stop and remove container
   docker stop postgres-db
   docker rm postgres-db
   
   # Recreate container with same volume
   docker run -d --name postgres-db-new \
     -e POSTGRES_PASSWORD=password \
     -e POSTGRES_DB=myapp \
     -v postgres-data:/var/lib/postgresql/data \
     postgres:13-alpine
   
   # Verify data persistence
   docker exec postgres-db-new psql -U postgres -d myapp -c "SELECT * FROM test;"
   ```

### Exercise 3: Bind Mounts

1. **Create development environment:**
   ```bash
   # Create project directory
   mkdir -p ~/docker-projects/my-app
   cd ~/docker-projects/my-app
   
   # Create application files
   cat << EOF > index.html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My App</title>
   </head>
   <body>
       <h1>Hello from Docker!</h1>
       <p>This is a bind mount example.</p>
   </body>
   </html>
   EOF
   
   # Create Dockerfile
   cat << EOF > Dockerfile
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   EOF
   ```

2. **Run container with bind mount:**
   ```bash
   # Build image
   docker build -t my-app .
   
   # Run with bind mount for development
   docker run -d --name dev-app \
     -p 8080:80 \
     -v $(pwd):/usr/share/nginx/html \
     my-app
   
   # Test the application
   curl http://localhost:8080
   ```

3. **Test live code changes:**
   ```bash
   # Modify the HTML file
   echo "<p>Updated content!</p>" >> index.html
   
   # Check if changes are reflected (no restart needed)
   curl http://localhost:8080
   ```

### Exercise 4: Anonymous Volumes

1. **Create containers with anonymous volumes:**
   ```bash
   # Create container with anonymous volume
   docker run -d --name redis-cache \
     -v /data \
     redis:alpine
   
   # Create container with multiple anonymous volumes
   docker run -d --name multi-volume-app \
     -v /app/data \
     -v /app/logs \
     -v /app/config \
     nginx:alpine
   ```

2. **Inspect anonymous volumes:**
   ```bash
   # Find anonymous volumes
   docker volume ls --filter "dangling=true"
   
   # Get volume details
   docker inspect redis-cache --format='{{.Mounts}}'
   ```

### Exercise 5: Volume Backup and Restore

1. **Create backup of volume data:**
   ```bash
   # Create test data
   docker exec postgres-db-new psql -U postgres -d myapp -c "INSERT INTO test (name) VALUES ('backup test');"
   
   # Create backup
   docker run --rm -v postgres-data:/data -v $(pwd):/backup \
     alpine tar czf /backup/postgres-backup.tar.gz -C /data .
   
   # Verify backup
   ls -la postgres-backup.tar.gz
   ```

2. **Restore from backup:**
   ```bash
   # Create new volume
   docker volume create postgres-restored
   
   # Restore data
   docker run --rm -v postgres-restored:/data -v $(pwd):/backup \
     alpine sh -c "cd /data && tar xzf /backup/postgres-backup.tar.gz"
   
   # Test restored data
   docker run -d --name postgres-restored-db \
     -e POSTGRES_PASSWORD=password \
     -e POSTGRES_DB=myapp \
     -v postgres-restored:/var/lib/postgresql/data \
     postgres:13-alpine
   
   # Verify restoration
   docker exec postgres-restored-db psql -U postgres -d myapp -c "SELECT * FROM test;"
   ```

### Exercise 6: Advanced Volume Management

1. **Create volume with specific options:**
   ```bash
   # Create volume with size limit (if supported)
   docker volume create --opt size=1g my-limited-volume
   
   # Create volume with specific mount options
   docker volume create --opt type=tmpfs --opt device=tmpfs --opt o=size=100m my-tmpfs-volume
   ```

2. **Use volumes with Docker Compose:**
   ```bash
   # Create docker-compose.yml
   cat << EOF > docker-compose.yml
   version: '3.8'
   services:
     web:
       image: nginx:alpine
       ports:
         - "8080:80"
       volumes:
         - web-data:/usr/share/nginx/html
         - ./config:/etc/nginx/conf.d
     
     db:
       image: postgres:13-alpine
       environment:
         POSTGRES_PASSWORD: password
         POSTGRES_DB: myapp
       volumes:
         - db-data:/var/lib/postgresql/data
   
   volumes:
     web-data:
       driver: local
     db-data:
       driver: local
   EOF
   
   # Start services
   docker-compose up -d
   
   # Check volumes
   docker volume ls
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Multi-Container Data Sharing

1. **Create a shared data architecture:**
   ```bash
   # Create shared volume
   docker volume create shared-data
   
   # Create producer container
   docker run -d --name data-producer \
     -v shared-data:/shared \
     alpine sh -c "while true; do echo \$(date) > /shared/data.txt; sleep 5; done"
   
   # Create consumer container
   docker run -d --name data-consumer \
     -v shared-data:/shared \
     alpine sh -c "while true; do cat /shared/data.txt; sleep 2; done"
   
   # Create backup container
   docker run -d --name data-backup \
     -v shared-data:/shared \
     -v $(pwd):/backup \
     alpine sh -c "while true; do cp /shared/data.txt /backup/backup-\$(date +%s).txt; sleep 10; done"
   ```

2. **Monitor data sharing:**
   ```bash
   # Check logs from all containers
   docker logs data-producer
   docker logs data-consumer
   docker logs data-backup
   
   # Check shared volume
   docker run --rm -v shared-data:/shared alpine ls -la /shared
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you create and manage named volumes?
- [ ] Do you understand bind mounts for development?
- [ ] Can you implement data persistence strategies?
- [ ] Can you backup and restore volume data?
- [ ] Do you understand volume types and use cases?
- [ ] Can you configure multi-container data sharing?

### Skills Demonstrated:
- ✅ Volume type understanding
- ✅ Named volume management
- ✅ Bind mount implementation
- ✅ Data persistence strategies
- ✅ Backup and restore procedures
- ✅ Multi-container data sharing

## 🔍 Troubleshooting

### Common Issues:

1. **Volume not mounting:**
   ```bash
   # Check volume exists
   docker volume ls
   
   # Check mount point
   docker inspect <container-name> --format='{{.Mounts}}'
   
   # Check volume permissions
   docker run --rm -v <volume-name>:/data alpine ls -la /data
   ```

2. **Data not persisting:**
   ```bash
   # Check volume usage
   docker volume inspect <volume-name>
   
   # Verify data in volume
   docker run --rm -v <volume-name>:/data alpine find /data -type f
   
   # Check container mount
   docker inspect <container-name> --format='{{.Mounts}}'
   ```

3. **Permission issues:**
   ```bash
   # Check volume ownership
   docker run --rm -v <volume-name>:/data alpine ls -la /data
   
   # Fix permissions
   docker run --rm -v <volume-name>:/data alpine chown -R 1000:1000 /data
   ```

## 📚 Additional Resources

- [Docker Volumes](https://docs.docker.com/storage/volumes/)
- [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
- [Volume Drivers](https://docs.docker.com/storage/volumes/#use-a-volume-driver)

## 🎉 Lab Completion

Congratulations! You've completed the Data Management with Volumes lab. You now understand:
- Docker volume types and use cases
- Named volume creation and management
- Bind mounts for development workflows
- Data persistence strategies
- Volume backup and restore procedures

**Next Lab**: [Lab 06: Docker Compose Basics](../docker/lab-06-docker-compose-basics.md)

---

**Happy Learning! 🐳** 