
# **Day 2: Basic Docker Commands - A Beginner’s Guide**

### **Objective**:
Learn essential Docker commands and practice using them to interact with containers.

---

## **Introduction to Basic Docker Commands**

Docker's command-line interface (CLI) is a powerful tool that allows you to interact with Docker by managing images, containers, networks, and volumes. Today, you’ll focus on learning and practicing some of the most commonly used commands to control Docker containers.

### **Essential Commands Covered Today**:
- `docker run`
- `docker ps`
- `docker stop`
- `docker rm`

---

## **Step 1: Running a Simple Container**

The `docker run` command is one of the most fundamental commands in Docker. It creates and runs a container based on a specified image.

### **Run a Simple Container**:

You will use the `hello-world` image again as it’s a small, pre-built image that is great for testing.

1. Open your terminal or Docker CLI.
2. Use the following command to run the `hello-world` image:

```bash
docker run hello-world
```

**Explanation**:
- `docker run` tells Docker to run a container based on the specified image (`hello-world` in this case).
- If the image isn't available locally, Docker automatically pulls it from Docker Hub.
- The output from this command should display the message "Hello from Docker!" to indicate that Docker is working correctly.

### **Running a Web Server Container**:

Let's try running a simple container with a web server. We will use the official `nginx` image from Docker Hub:

```bash
docker run -d -p 8080:80 nginx
```

**Explanation**:
- `-d`: Runs the container in detached mode (in the background).
- `-p 8080:80`: Maps port 8080 on your local machine to port 80 inside the container (where `nginx` runs by default).

To check if `nginx` is running, open your browser and go to `http://localhost:8080`. You should see the default Nginx welcome page.

---

## **Step 2: Listing Running Containers**

The `docker ps` command is used to list all running containers.

### **List Running Containers**:

Run the following command to view your active containers:

```bash
docker ps
```

The output shows columns like **CONTAINER ID**, **IMAGE**, **STATUS**, and **PORTS**, providing a snapshot of all currently running containers.

### **Listing All Containers (Stopped and Running)**:

If you want to see all containers, including those that have stopped:

```bash
docker ps -a
```

---

## **Step 3: Stopping a Container**

Sometimes, you need to stop a running container. The `docker stop` command gracefully stops a container by giving it time to complete ongoing operations.

### **Stop a Running Container**:

1. Identify the container ID or name by running `docker ps`.
2. Use the `docker stop` command:

```bash
docker stop <container_id>
```

Replace `<container_id>` with the actual container ID from the `docker ps` output.

---

## **Step 4: Removing Containers**

Docker doesn't automatically remove containers after they are stopped. If you want to free up system resources or clean up old containers, use the `docker rm` command.

### **Remove a Stopped Container**:

1. After stopping a container, you can remove it by running:

```bash
docker rm <container_id>
```

If you want to remove multiple containers, you can pass multiple container IDs:

```bash
docker rm <container_id_1> <container_id_2>
```

If a container is still running, you must stop it first before removing it.

---

## **Step 5: Docker CLI Cheat Sheet**

Docker commands can be extensive and complex. Fortunately, there are many references available to help you. The **Docker CLI Cheat Sheet** is an excellent resource to keep handy.

You can find the official Docker CLI cheat sheet here:
- **[Docker CLI Cheat Sheet](https://docs.docker.com/get-started/docker_cheat_sheet.pdf)**

This document provides quick references for commonly used Docker commands.

---

## **Step 6: Experimenting with Pre-built Images from Docker Hub**

Docker Hub is a repository for Docker images where you can find images created by others and use them for your containers. Let's practice by running a few more pre-built images:

### **Run an Ubuntu Container**:

To experiment with Ubuntu in a container, use this command:

```bash
docker run -it ubuntu bash
```

**Explanation**:
- `-it`: Enables interactive mode, allowing you to interact with the container.
- `ubuntu`: Specifies the Ubuntu image.
- `bash`: Runs the Bash shell inside the container.

You now have an interactive Ubuntu session inside a container. Type `exit` to leave the container.

### **Running a Redis Container**:

Redis is an in-memory database. To run Redis in a container, try:

```bash
docker run -d redis
```

**Explanation**:
- `redis`: This specifies the image for Redis.
- `-d`: Runs it in detached mode, in the background.

You can verify it's running with `docker ps`, and stop it with `docker stop <container_id>`.

---

## **Summary of Commands**

Here’s a quick overview of the commands covered today:

| **Command**               | **Description** |
|---------------------------|-----------------|
| `docker run <image>`       | Runs a container based on the image. |
| `docker ps`                | Lists all running containers. |
| `docker ps -a`             | Lists all containers (running and stopped). |
| `docker stop <container_id>`| Stops a running container. |
| `docker rm <container_id>` | Removes a stopped container. |
| `docker run -d <image>`    | Runs a container in detached mode (in the background). |
| `docker run -it <image> bash` | Runs a container in interactive mode with the Bash shell. |

---

## **Additional Resources**

- **[Docker Hub](https://hub.docker.com/)**: The central repository for finding pre-built Docker images.
- **[Docker Documentation](https://docs.docker.com/)**: Detailed instructions on all Docker commands and features.
- **[Play with Docker](https://labs.play-with-docker.com/)**: An online playground to experiment with Docker without installing anything on your machine.

By practicing these commands, you’ll build a strong foundation for working with Docker containers, which is essential for deploying and managing applications in various environments.
