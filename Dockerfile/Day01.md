
# Day 1: Introduction to Dockerfiles

Today, we will focus on understanding the fundamental structure and purpose of Dockerfiles. This foundational knowledge will enable you to create and manage Docker images effectively.

## Objective

- Gain a clear understanding of what a Dockerfile is and its role in containerization.
- Learn the basic syntax and structure of a Dockerfile.
- Familiarize yourself with common Dockerfile instructions.

## Topics

### What is a Dockerfile?

A Dockerfile is a text document that contains a set of instructions for building a Docker image. It provides the blueprint for creating containerized applications by specifying the base image, dependencies, and configurations needed.

### Basic Syntax and Structure

A typical Dockerfile consists of several instructions, each performing a specific task. These instructions are executed in order, from top to bottom.

### Common Instructions

- **`FROM`**: Specifies the base image from which you are building. This is usually the first instruction in any Dockerfile.

```
  FROM ubuntu:20.04
```

- **`RUN`**: Executes commands inside the container during the image build process. It is used to install packages or perform setup tasks.

  ```
  RUN apt-get update && apt-get install -y python3
  ```

- **`CMD`**: Provides default commands for an executing container. It can be overridden at runtime.

  ```
  CMD ["python3", "app.py"]
  ```

- **`COPY`**: Copies files or directories from the host machine into the image.

  ```
  COPY . /app
  ```

- **`ADD`**: Similar to `COPY`, but also supports URL sources and automatic extraction of compressed files.

  ```
  ADD https://example.com/file.tar.gz /app/
  ```

- **`EXPOSE`**: Informs Docker that the container listens on specified network ports at runtime. It does not actually publish the port; it is primarily for documentation purposes.

  ```
  EXPOSE 80
  ```

## Activities

### Create a Simple Dockerfile for a Basic Application

1. Create a new directory for your project.
2. Inside this directory, create a file named `Dockerfile`.
3. Write a simple Dockerfile using the instructions covered:

    ```
    FROM python:3.9-slim
    WORKDIR /app
    COPY . /app
    RUN pip install --no-cache-dir -r requirements.txt
    EXPOSE 5000
    CMD ["python", "app.py"]
    ```

### Build and Run a Docker Image Using the Created Dockerfile

1. Open your terminal and navigate to the directory containing your `Dockerfile`.
2. Build the Docker image:

    ```
        docker build -t my-simple-app .
    ```

3. Run a container from the built image:

    ```
    docker run -p 5000:5000 my-simple-app
    ```
