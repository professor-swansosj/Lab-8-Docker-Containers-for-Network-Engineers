# Lab 8 — Docker Containers for Network Engineers

**Course:** Software Defined Networking  
**Module:** Network Automation Fundamentals • **Lab #:** 8  
**Estimated Time:** 120–150 minutes

## Repository structure

```text
Lab-8-Docker-Containers-for-Network-Engineers
├── .devcontainer
│   └── devcontainer.json
├── .gitignore
├── .markdownlint.json
├── .markdownlintignore
├── .pettierrc.yml
├── INSTRUCTIONS.backup.md
├── INSTRUCTIONS.md
├── LICENSE
├── README.backup.md
├── README.md
├── data
│   └── inventory.example.yml
├── lab.yml
├── prettierrc.yml
├── requirements.txt
└── src
    ├── __init__.py
    └── main.py
```


## Lab Topics

### What is a Docker Container?
A Docker container is a lightweight, portable, and self-sufficient unit that encapsulates an application and all its dependencies, libraries, and configuration files. Containers are built from Docker images, which serve as the blueprint for the container's filesystem and environment.

Key characteristics of Docker containers include:
- **Isolation**: Containers run in their own isolated environments, ensuring that applications do not interfere with each other.
- **Portability**: Containers can run consistently across different environments, such as development, testing, and production.
- **Efficiency**: Containers share the host system's kernel, making them more lightweight and faster to start than traditional virtual machines.

In our last lab, we explored the basics of Docker and containerization. In this lab, we will dive deeper into building and running Docker containers for network engineering tasks. Below is a simple example of how to use Docker to run a container.


```bash
docker run hello-world

```

### Containers vs Virtual Machines
While both containers and virtual machines (VMs) provide isolated environments for running applications, they do so in fundamentally different ways.

**Virtual Machines**:
- VMs run a full operating system (OS) on top of a hypervisor, which abstracts the underlying hardware.
- Each VM includes its own OS kernel, libraries, and applications, leading to larger resource consumption.
- VMs are typically slower to start and require more disk space.

**Containers**:
- Containers share the host OS kernel and isolate applications at the process level.
- They package only the application and its dependencies, making them more lightweight and efficient.
- Containers start quickly and use less disk space compared to VMs.


### Containers in Network Engineering
Containers have become increasingly popular in network engineering for several reasons:
- **Automation**: Containers can be easily integrated into CI/CD pipelines, enabling automated testing and deployment of network applications.
- **Scalability**: Containers can be quickly scaled up or down to meet changing network demands.
- **Consistency**: Containers ensure that applications run consistently across different environments, reducing the "it works on my machine" problem.
- **Microservices**: Containers facilitate the development and deployment of microservices architectures, allowing network functions to be broken down into smaller, manageable components.

In this lab, we will explore how to leverage Docker containers to build a FastAPI application that interacts with network devices using Ansible.


### The Dockerfile
A Dockerfile is a text file that contains a series of instructions for building a Docker image. It defines the base image, application code, dependencies, environment variables, and commands to run when the container starts.

Here is a simple example of a Dockerfile for a FastAPI application:


```dockerfile
# Use the official Python image as a base
FROM python:3.11

# Set the working directory
WORKDIR /app

# Copy the application code
COPY . .

# Install dependencies
RUN pip install -r requirements.txt

# Expose the application port
EXPOSE 8000

# Start the FastAPI application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

```

### Understanding Containers and Images
In summary, Docker containers are lightweight, portable units that encapsulate applications and their dependencies. They differ from virtual machines in their architecture and resource efficiency. Containers are widely used in network engineering for automation, scalability, and consistency.

The Dockerfile is a crucial component in the containerization process, as it defines how to build a Docker image. By following best practices in writing Dockerfiles, we can create efficient and maintainable images for our applications.

In the following sections of this lab, we will apply these concepts by building and running a Docker container for a FastAPI application that interacts with network devices using Ansible.


### Managing your Containers and Images
Once you have built and run your Docker containers, it's important to know how to manage them effectively. Here are some common commands for managing Docker containers and images:

**Managing Containers**:
- `docker ps`: List all running containers.
- `docker ps -a`: List all containers (running and stopped).
- `docker stop <container_id>`: Stop a running container.
- `docker start <container_id>`: Start a stopped container.
- `docker restart <container_id>`: Restart a container.
- `docker rm <container_id>`: Remove a stopped container.
- `docker logs <container_id>`: View the logs of a container.

**Managing Images**:
- `docker images`: List all Docker images on your system.
- `docker rmi <image_id>`: Remove a Docker image.
- `docker tag <image_id> <new_image_name>:<tag>`: Tag an image with a new name and tag.
- `docker pull <image_name>`: Pull an image from Docker Hub or another registry.
- `docker push <image_name>`: Push an image to Docker Hub or another registry.

By mastering these commands, you can efficiently manage your Docker containers and images, ensuring that your development and deployment processes run smoothly.




## License
© 2025 Your Name — Classroom use.
