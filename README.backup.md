# Lab 8 Docker Containers for Network Engineers

## :triangular_flag_on_post: Learning Goals
- **Understand what Docker is** and why containers are useful in modern IT and network automation
- **Differentiate containers from virtual machines**, focusing on efficiency, portability, and resource usage
- **Explain the benefits of containerization** for network engineers, including reproducible environments, simplified deployments, and lightweight microservices
- **Run a basic container** (the Docker “Hello World” example) to validate installation and build confidence with Docker commands
- **Explore Dockerfiles** as the blueprint for building custom containers
- **Build a containerized FastAPI application**, reinforcing concepts of microservices and API-driven automation
- **Create multiple FastAPI endpoints**, including:
    - One that returns ASCII art with a word or phrase chosen by the user (showcasing simple text processing)
    - One that performs a network device backup (tying Docker back to real-world network engineering use cases)
    - One additional endpoint (to be finalized) that demonstrates extensibility—such as returning system stats, pinging a host, or generating a small device report
- **Connect Docker to previous labs** by calling these FastAPI endpoints with `cURL`, showing how containerized APIs can integrate with automation workflows
- **Appreciate the role of containers* in scaling automation**, as a stepping stone toward orchestration platforms like Kubernetes

## :globe_with_meridians: Overview:
In this lab, you’ll be introduced to **Docker**, a platform that uses containers to package applications and their dependencies into lightweight, portable units. Unlike virtual machines, which emulate entire operating systems, containers share the host system’s kernel, making them faster to start, more efficient with resources, and easier to move between environments. For network engineers, Docker offers a way to create reproducible labs, run automation tools in isolated environments, and deploy microservices like APIs with minimal overhead.

You’ll begin by learning what Docker is, how it compares to traditional virtual machines, and why containers have become the standard for modern infrastructure and automation workflows. After validating your setup by running Docker’s “Hello World” container, you’ll progress to building your own container image using a **Dockerfile**. Inside this container, you’ll run a simple **FastAPI** application with multiple endpoints: one that prints ASCII art with a custom phrase, another that performs a network device backup, and an additional endpoint that demonstrates how easily containerized services can be extended. By the end of the lab, you’ll understand how containers fit into the automation landscape and have hands-on experience creating and running your own containerized application.

### What is Docker?
Docker is a platform that makes it easy to build, package, and run applications in containers. A container is a lightweight environment that bundles an application together with everything it needs to run—like libraries, dependencies, and configuration—so it behaves the same way no matter where you launch it. This solves the classic “it works on my machine” problem, because the container carries its environment with it.

Unlike a traditional virtual machine, a Docker container doesn’t run an entire guest operating system. Instead, it shares the host system’s kernel while keeping the application isolated. This makes containers start in seconds, use fewer resources, and scale up or down quickly. For network engineers, Docker provides a way to stand up lab environments, test automation tools, and even run microservices such as APIs or monitoring agents—all without needing heavy infrastructure. It’s a building block for modern automation pipelines and a stepping stone to larger orchestration tools like Kubernetes.

### Containers vs. Virtual Machines
At first glance, containers and virtual machines (VMs) seem similar—they both isolate applications from the underlying hardware and let you run multiple environments on a single system. The key difference lies in how they achieve that isolation.

A virtual machine runs a full operating system on top of a hypervisor, which means each VM has its own kernel and system resources. This makes VMs highly flexible, but also heavy: they take longer to boot, consume more memory and storage, and can be slower to scale.

A container, on the other hand, shares the host’s operating system kernel and only includes the application plus its dependencies. This makes containers far more lightweight—they start in seconds, use fewer resources, and can be packed densely on the same hardware. For example, you might only run a handful of VMs on a laptop, but you could easily spin up dozens of containers.

In practice, VMs are great when you need complete isolation or different operating systems, while containers shine for **portability**, **efficiency**, and **microservices**. For network engineers, this means containers are a faster way to deploy automation tools and APIs without waiting for full-blown virtual machines to boot.

### Benefits of Containers for Network Engineers
For network engineers, containers bring several practical advantages that make automation and lab work easier:

- **Consistency across environments** – A container behaves the same on your laptop, in a lab server, or in the cloud. This eliminates surprises when moving code between environments.

- **Lightweight and fast** – Containers start in seconds and use fewer resources than virtual machines. You can run many small services (like APIs or monitoring tools) side by side without overloading your system.

- **Easy to share and reuse** – With Docker Hub or private registries, you can publish and pull container images the same way developers share code. This makes it simple to distribute automation tools across a team.

- **Reproducible labs** – Instead of rebuilding environments manually, you can spin up prebuilt containers for testing. For example, you could launch a containerized network simulator, monitoring stack, or custom FastAPI service in minutes.

- **Modular automation** – Containers fit neatly into an event-driven or microservices design. One container might back up device configs, another might serve an API, and a third might run scheduled compliance checks—all loosely coupled but working together.

By adopting containers, network engineers can work more like developers: packaging their automation into portable, reproducible units that scale and integrate into modern CI/CD and orchestration workflows.

### Running Your First Container: Hello World
The simplest way to get started with Docker is by running the **“Hello World” container**. This container is a tiny image provided by Docker that prints a confirmation message to the terminal. It’s designed to validate that your Docker installation is working and that you can successfully pull and run images from Docker Hub.

To check that Docker is installed and available, you can run:

```bast

docker --version

```

This will display the installed Docker version, confirming that the CLI is accessible.

Next, run the “Hello World” container:

```bash

docker run hello-world

```

When you launch this command, Docker will:

1. Check if the `hello-world` image exists locally.

2. If not, automatically download it from Docker Hub.

3. Create a new container from the image.

4. Run the program inside it, which prints a friendly message to your terminal.

This small exercise demonstrates several key concepts: how Docker pulls images, how containers are created from those images, and how execution inside a container is isolated from the host. Once you’ve seen “Hello from Docker!” in your terminal, you’ll know your Docker environment is ready for building and running custom containers.

### Understanding Dockerfiles and Images
Now that you’ve successfully run the Hello World container, you’ve seen how Docker uses an image as the blueprint for creating containers. An image is essentially a packaged application: it contains the program itself, the libraries it depends on, and a small operating system layer to run everything consistently. Every time you launch a container, Docker creates it from an image. For example, when you ran `docker run hello-world`, Docker pulled the `hello-world` image from Docker Hub and spun up a container from it.

So where do images come from? They’re built using Dockerfiles. A Dockerfile is just a text file with instructions that tell Docker how to assemble an image. Each line in the Dockerfile creates a new layer in the image, whether that’s setting a base operating system, installing software, copying files, or defining the command to run. For example, a very simple Dockerfile might look like this:

```dockerfile

FROM python:3.11-slim
COPY app/ /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "main.py"]

```

In this case:

- `FROM` tells Docker to start from an existing Python image.

- `COPY` moves your application files into the image.

- `WORKDIR` sets the working directory inside the container.

- `RUN` installs dependencies.

- `CMD` defines what command should run when the container starts.

By writing a Dockerfile, you define everything your application needs in a repeatable and portable way. Later in this lab, you’ll create your own Dockerfile to containerize a FastAPI application—giving you a clear path from the “Hello World” demo to building your own custom network automation tools.

### Building a Custom FastAPI Container
To containerize an API, you’ll pair a small FastAPI app with a Dockerfile that defines its runtime. The idea is simple: your app exposes a few HTTP endpoints, and the Docker image bakes in Python, your dependencies, and a process manager (typically `uvicorn`) to serve it. A minimal FastAPI app might declare routes like `/ascii/{text}` and `/backup`, returning JSON or text. Keep the app self-contained—no global state, clear function boundaries, environment variables for secrets/targets—and return predictable responses so it’s easy to test with `curl`. For example, a FastAPI sketch could look like:

```python

from fastapi import FastAPI
app = FastAPI()

@app.get("/ascii/{text}")
def ascii_art(text: str):
    # transform text -> ascii art (placeholder)
    return {"art": f"* {text} *"}

@app.post("/backup")
def backup_device():
    # call your backup routine (placeholder)
    return {"status": "queued"}

```

On the image side, a Dockerfile typically starts from a slim Python base, copies only what you need, installs requirements, and sets a default command to run uvicorn. The key is keeping layers lean and reproducible (pin versions, avoid dev-only packages, and don’t copy your whole repo if you only need an `app/` directory). A sketch (intentionally incomplete) might resemble:

```dockerfile

FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ .
# Expose an internal port and run uvicorn (placeholder)
# CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

```

This pattern gives you a portable microservice: build once, run anywhere. From here you can add routes for your ASCII output, a network backup action, and one more endpoint of your choosing, then verify everything with targeted `curl` calls.

### Creating API Endpoints in FastAPI
An endpoint in FastAPI is just a Python function tied to a specific route (URL path) and HTTP method. You decorate the function with @app.get() or @app.post() to tell FastAPI when it should run. Each endpoint receives a request, processes it, and returns a response—usually JSON or plain text.

For example, a small endpoint might look like this:

```python

@app.get("/hello")
def say_hello():
    return {"message": "Hello, Network Engineers!"}

```

From here, you can extend the concept:

- An ASCII art endpoint could take a string parameter and transform it into ASCII (perhaps using a tool like `figlet`).

- A backup endpoint could trigger a function that connects to a device, retrieves its configuration, and returns a status.

- A third endpoint might expose something fun or useful—like returning system stats, showing container uptime, or pulling an API joke—demonstrating how easy it is to expand your service.

Each of these endpoints should be lightweight and single-purpose. The power of FastAPI is that you can quickly add new routes, wire them to Python logic, and expose them in a consistent way—all inside the same container. Over time, this approach grows into a microservice: a containerized app with multiple automation tools living behind one simple API.

#### - ASCII Art Endpoint
One of the simplest but most fun ways to demonstrate a custom API is by turning plain text into **ASCII art**. The idea is that your endpoint accepts a word or phrase as input (through the URL or query parameters), passes that string through an ASCII generator, and returns the result. This shows how an API can wrap even the most basic command-line tool or Python function into a service you can call from anywhere.

For example, if you had an endpoint like `/ascii/Automation` and hit it with `curl`, the response might look like:

```markdown

   _              _   _             
  / \   _ __   __| | (_)_   _____   
 / _ \ | '_ \ / _` | | \ \ / / _ \  
/ ___ \| | | | (_| | | |\ V /  __/  
/_/   \_\_| |_|\__,_| |_| \_/ \___|  


```

Behind the scenes, this could be done by calling a tool like `figlet` or by using a simple Python library that generates ASCII art. The point isn’t the tool itself—it’s seeing how easily you can expose local functionality through a web API. This endpoint teaches you how FastAPI can take an input, process it, and return a dynamic response. It also sets the stage for more advanced endpoints that interact with real devices.

#### - Network Device Backup Endpoint
A highly practical endpoint to include is one that performs a network device backup. The purpose is simple: when this endpoint is triggered, it connects to a device, retrieves the running configuration, and either returns it in the response or stores it for later use. This highlights how a containerized API can wrap common network engineering tasks and make them accessible with a single HTTP call.

How you implement the backup is flexible. The endpoint could call a Python script that uses a library like Netmiko, ncclient, or the RESTCONF/NETCONF APIs you’ve already worked with. Alternatively, it could kick off an Ansible playbook that handles the backup process and then return the result. Both approaches show how you can integrate existing automation tools into a clean API interface.

For example, a `/backup` endpoint might respond with something like:

```json

{
  "device": "router1",
  "status": "backup complete",
  "timestamp": "2025-09-07T14:33:00Z"
}

```

This kind of endpoint demonstrates how Docker, FastAPI, and automation tools (Python or Ansible) can work together to make routine operational tasks repeatable, portable, and easy to consume.

#### - Extending with an Additional Endpoint
Once you’ve built endpoints for ASCII art and device backups, it’s easy to imagine adding more. The power of FastAPI is that each new endpoint is just another small function tied to a route. This makes your container feel like a toolbox—one container, many services.

An additional endpoint could demonstrate a new idea without being too complex. For example:

- A `/status` endpoint that returns basic system information from inside the container (uptime, memory, disk).

- A `/ping/{host}` endpoint that takes a hostname or IP and returns whether it’s reachable.

- A `/report` endpoint that calls one of your earlier lab scripts (like pulling a joke, drawing a card, or generating a device summary).

The point isn’t which task you choose—it’s showing that you can quickly extend your API by wiring existing automation or utilities into new routes. This flexibility mirrors how production microservices grow: starting small, then expanding endpoint by endpoint as new needs arise. It reinforces the lesson that containers let you package these services into portable, reproducible tools for network automation.

### Calling Your Containerized API with cURL
Once your FastAPI container is running, you can interact with it using the same tool you’ve been practicing with throughout the course: cURL. Each FastAPI endpoint is just an HTTP route, which means you can test them by sending requests directly from the command line. This step ties everything together—you’ve built a container, defined endpoints, and now you’ll confirm they behave as expected when called externally.

For example, if your container is serving on port 8000, you could run:

```bash

curl http://localhost:8000/ascii/Automation

```

to hit your ASCII art endpoint, or:

```bash

curl -X POST http://localhost:8000/backup

```

to trigger the device backup. The responses will be returned as plain text or JSON, depending on how you designed the endpoint.

This process reinforces the idea that containers aren’t just isolated environments—they expose consistent, portable services to the outside world. By testing your containerized API with cURL, you’re validating that your automation can be packaged, deployed, and consumed just like any other microservice. This is the same model used in enterprise environments, scaled up across fleets of containers.

---

## :card_file_box: File Structure:

'''
file structure
'''

---

## Components
text

### 1. **Component 1**
text

### 2. **Component 2**
text

### 3. **Component 3**
text

## :memo: Instructions
1. text
2. text
3. text

## :page_facing_up: Logging
text

## :green_checkmark: Grading Breakdown
- x pts: 
- x pts:
- x pts: