# Detailed Markdown content for Day 6 of the Docker series
docker_day6_markdown_content = """
# Day 6: Data Management with Volumes

## Objective:
Learn how to persist data in containers using volumes.

## Activities:
1. **Understand the Difference Between Bind Mounts and Volumes**:
    - Bind Mounts
    - Volumes

2. **Create Volumes and Attach Them to Containers**:
    - Creating a volume.
    - Attaching a volume to a container.

3. **Practice Data Persistence by Running a Database Container with a Volume**:
    - Setting up a database container.
    - Attaching a volume to the database container.
    - Verifying data persistence.

---

## Bind Mounts vs Volumes

### Bind Mounts
Bind mounts can be used to mount a host directory or file into a container. They provide a direct link between the host filesystem and the container.

#### Example of Bind Mount:

    docker run -d --name my_container -v /path/on/host:/path/in/container nginx


### Volumes
Volumes are the preferred way to persist data in Docker. They are managed by Docker and can be shared between containers.

#### Creating and Using a Volume:

    docker volume create my_volume
    docker run -d --name my_container -v my_volume:/path/in/container nginx


---

## Creating Volumes and Attaching Them to Containers

### Creating a Volume:

    docker volume create my_data_volume


### Attaching a Volume to a Container:

    docker run -d --name my_app -v my_data_volume:/usr/share/nginx/html nginx


---

## Data Persistence with a Database Container

### Setting Up a Database Container:
Run a MySQL container with a volume to ensure data persistence.

#### Running the MySQL Container:

    docker run -d \\
    --name my_mysql \\
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \\
    -v my_db_volume:/var/lib/mysql \\
    mysql:latest


### Verifying Data Persistence:
1. Connect to the MySQL container:
  

    docker exec -it my_mysql mysql -p


2. Create a database and add some data:
   

    CREATE DATABASE testdb;
    USE testdb;
    CREATE TABLE users (id INT AUTO_INCREMENT, name VARCHAR(100), PRIMARY KEY(id));
    INSERT INTO users (name) VALUES ('John Doe');
   

3. Stop and remove the container:
  

    docker stop my_mysql
    docker rm my_mysql


4. Run the MySQL container again with the same volume:


    docker run -d \\
    --name my_mysql \\
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \\
    -v my_db_volume:/var/lib/mysql \\
    mysql:latest


5. Connect to the MySQL container and verify the data:

        docker exec -it my_mysql mysql -p
    
    
     
       USE testdb;
       SELECT * FROM users;

---

## Additional Resources:
- [Docker Storage Overview](https://docs.docker.com/storage/)
- [Managing Docker Volumes](https://docs.docker.com/storage/volumes/)
- [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)

---
