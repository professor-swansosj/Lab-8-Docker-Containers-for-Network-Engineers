# Lab Instructions

## Lab Title
**`Docker Containers for Network Engineers`**

## Version
`v1.0`

## Estimated Time
`120-150 Minutes`

---

## Learning Objectives
By the end of this lab, you will be able to:
- Objective 1 – Run your first Docker container using the hello-world image to validate your Docker environment.
- Objective 2 – Understand the difference between Docker images and containers through hands-on exploration.
- Objective 3 – Create a Dockerfile to build custom container images with specific dependencies and configurations.
- Objective 4 – Containerize a FastAPI application with multiple endpoints for network automation tasks.
- Objective 5 – Build and run a custom container that serves ASCII art and integrates with external APIs.
- Objective 6 – Extend your containerized API to include network device backup functionality using Ansible.
- Objective 7 – Test containerized services using cURL commands and validate API responses.
- Objective 8 – Version and manage Docker images using tags and understand container lifecycle management.

These objectives build foundational skills for aspiring **network engineers** and **infrastructure specialists** in modern containerized environments.

---

## Tools & Technologies
You will use:
- Editor/Runtime
    - Visual Studio Code (no Dev Container this time!)
    - Docker Desktop or Docker Engine
    - Git & GitHub Classroom repository
- Language & Libraries
    - Python 3.x
    - FastAPI
    - Uvicorn (ASGI server)
    - Requests library
    - Ansible (for network automation)
    - pyfiglet (for ASCII art generation)
- Target Services
    - Dad Jokes Public API
    - Deck of Cards Public API
    - Cisco DevNet Always-On Sandbox (for Ansible backup)
- Command Line
    - Docker CLI
    - cURL
    - Bash/Zsh

---

## Prerequisites
Before starting, make sure you:
- Have Docker installed and running on your local machine (not in a dev container).
- Understand basic Linux CLI: navigating directories, running commands, viewing files.
- Have Git & GitHub fundamentals: clone, commit, push.
- Know Python basics: functions, classes, dictionaries, HTTP requests, API development.
- Have completed previous FastAPI labs with Dad Jokes/Deck of Cards API integration.
- Are familiar with Ansible basics for network device automation.

---

## Deliverables
Commit and push the following to your Classroom repo:

1) **Docker Configuration Files**
- `Dockerfile`
    - Multi-stage build configuration for FastAPI application with all dependencies.
- `requirements.txt`
    - Python dependencies including FastAPI, uvicorn, requests, ansible, pyfiglet.
- `.dockerignore`
    - Excludes unnecessary files from Docker build context.

2) **Source Code (in `app/`)**
- `app/main.py`
    - FastAPI application with at least 3 endpoints: ASCII art, joke/card API, and network backup.
- `app/ansible_backup.py`
    - Module for triggering Ansible network device backups.
- `app/reverse_api.py`
    - Integration with Dad Jokes or Deck of Cards APIs from previous labs.

3) **Ansible Configuration (in `ansible/`)**
- `ansible/backup-playbook.yml`
    - Ansible playbook for backing up Cisco DevNet sandbox device.
- `ansible/inventory.yml`
    - Inventory file for target network devices.

4) **Data Artifacts (in `data/`)**
- `data/container_logs.txt`
    - Docker container runtime logs showing successful API calls.
- `data/api_responses/`
    - Sample JSON responses from each endpoint for validation.

5) **Documentation (in `docs/`)**
- `docs/container_usage.md`
    - Instructions for building, running, and testing the containerized application.
- `docs/api_endpoints.md`
    - Documentation of all available API endpoints with example cURL commands.

6) **Logs (in `logs/`)**
- `logs/lab8_docker.log`
    - Contains `LAB8_START`/`LAB8_END`, `DOCKER_HELLO_OK`, `IMAGE_BUILD_OK`, `CONTAINER_RUN_OK`, `API_TEST_OK`, and detailed endpoint testing results.
- `logs/build_output.log`
    - Docker build output showing successful image creation and layer information.

> ⚠️ **Important:** This lab runs Docker natively (not in a dev container) to demonstrate real containerization concepts. Autograding will validate container functionality, API responses, and Docker image properties.

---

## Logging Snippet to Include
> **INSERT THIS CODE SNIPPET AT THE VERY TOP OF YOUR MAIN.PY FILE RIGHT AFTER YOUR IMPORT STATEMENTS.**

```python
from datetime import datetime, timezone
import os

def now_iso():
    return datetime.now(timezone.utc).isoformat().replace("+00:00","Z")

def log(line, path):
    os.makedirs(os.path.dirname(path), exist_ok=True)
    with open(path, "a", encoding="utf-8") as f:
        f.write(f"{line}\n")

# usage:
# log(f"LAB8_START ts={now_iso()}", "logs/lab8_docker.log")
```

---

## Overview
In this lab, you will transition from **using** containers (dev containers) to **building** containers. You'll start with Docker's famous "Hello World" example to validate your setup, then progress to creating your own custom container that packages a FastAPI application with multiple network automation endpoints.

This lab represents the culmination of your network automation journey—taking the APIs, Ansible playbooks, and Python scripts you've built throughout the course and packaging them into a portable, reproducible container that can run anywhere Docker is available.

💡 **Why this matters:** Containerization is the foundation of modern application deployment and scaling. As a network engineer, understanding how to package your automation tools into containers makes them portable across environments, easier to share with teams, and ready for orchestration platforms like Kubernetes. This lab bridges the gap between writing automation scripts and deploying them in production environments.

---

## Instructions

Follow these steps in order:

### Step 1 – Setup Your Local Docker Environment
**What you're doing:** Ensuring Docker is installed and accessible on your local machine (not in a dev container).

1. **Verify Docker Installation**
```bash
docker --version
docker info
```

2. **Clone and Enter Repository**
```bash
git clone <repo-url>
cd <repo-name>
```

3. **Create Initial Directory Structure**
```bash
mkdir -p app ansible data/{api_responses} docs logs
```

**LOG REQUIREMENT:** `LAB8_START ts=<timestamp>`

**Done when:**
- `docker --version` shows Docker version information.
- `docker info` displays Docker daemon details without errors.
- Repository is cloned and directory structure is created.
- `logs/lab8_docker.log` contains the start timestamp.

### Step 2 – Run Your First Container
**What you're doing:** Validating your Docker setup with the classic "Hello World" container.

1. **Pull and Run Hello World**
```bash
docker run hello-world
```

2. **Explore What Happened**
- Docker automatically downloaded the hello-world image
- Created a container from that image
- Ran the program inside the container
- Container exited after printing the welcome message

3. **Verify the Process**
```bash
docker ps -a          # See all containers (including stopped ones)
docker images          # See downloaded images
```

**LOG REQUIREMENT:** `DOCKER_HELLO_OK ts=<timestamp> output="Hello from Docker!"`

**Done when:**
- Terminal displays "Hello from Docker!" message.
- `docker images` shows the hello-world image.
- `docker ps -a` shows the completed hello-world container.
- Log entry confirms successful hello-world execution.

### Step 3 – Build Your FastAPI Application
**What you're doing:** Creating the FastAPI application that you'll later containerize.

1. **Create Requirements File**
Create `requirements.txt`:
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
requests==2.31.0
pyfiglet==1.0.2
ansible==8.5.0
```

2. **Build Main FastAPI Application**
Create `app/main.py` with the logging snippet and these endpoints:
- `GET /` - Health check endpoint
- `GET /ascii/{text}` - Convert text to ASCII art
- `GET /joke` - Fetch a random dad joke
- `GET /card` - Draw a random card from deck
- `POST /backup` - Trigger network device backup

3. **Create Supporting Modules**
- `app/reverse_api.py` - Integration with Dad Jokes/Cards APIs
- `app/ansible_backup.py` - Ansible integration for device backups

**Example FastAPI Structure:**
```python
from fastapi import FastAPI
import pyfiglet
import requests
from .reverse_api import get_dad_joke, draw_card
from .ansible_backup import trigger_backup

app = FastAPI(title="Network Engineer's Toolkit", version="1.0.0")

@app.get("/")
def health_check():
    return {"status": "healthy", "message": "Network Automation API is running"}

@app.get("/ascii/{text}")
def create_ascii_art(text: str):
    ascii_art = pyfiglet.figlet_format(text)
    return {"text": text, "ascii": ascii_art}
```

**LOG REQUIREMENT:** `FASTAPI_APP_CREATED ts=<timestamp> endpoints=4`

**Done when:**
- `app/main.py` contains at least 4 functional endpoints.
- Supporting modules are created and imported correctly.
- Application can be tested locally with `uvicorn app.main:app --reload`.

### Step 4 – Create Ansible Network Backup Integration
**What you're doing:** Adding network automation functionality to your containerized API.

1. **Create Ansible Playbook**
Create `ansible/backup-playbook.yml`:
```yaml
---
- name: Backup Cisco Device Configuration
  hosts: cisco_devices
  gather_facts: no
  tasks:
    - name: Backup running configuration
      ios_config:
        backup: yes
        backup_options:
          filename: "{{ inventory_hostname }}_config.txt"
          dir_path: "./backups"
```

2. **Create Inventory File**
Create `ansible/inventory.yml` with Cisco DevNet Always-On sandbox details:
```yaml
cisco_devices:
  hosts:
    sandbox-iosxe-latest-1.cisco.com:
      ansible_host: sandbox-iosxe-latest-1.cisco.com
      ansible_user: developer
      ansible_password: C1sco12345
      ansible_network_os: ios
      ansible_connection: network_cli
```

3. **Integrate Ansible into FastAPI**
Update `app/ansible_backup.py`:
```python
import subprocess
import json
from datetime import datetime

def trigger_backup():
    try:
        result = subprocess.run([
            "ansible-playbook", 
            "ansible/backup-playbook.yml", 
            "-i", "ansible/inventory.yml"
        ], capture_output=True, text=True, cwd="/app")
        
        return {
            "status": "success" if result.returncode == 0 else "failed",
            "timestamp": datetime.utcnow().isoformat(),
            "output": result.stdout,
            "errors": result.stderr if result.returncode != 0 else None
        }
    except Exception as e:
        return {"status": "error", "message": str(e)}
```

**LOG REQUIREMENT:** `ANSIBLE_INTEGRATION_OK ts=<timestamp> playbook=backup-playbook.yml`

**Done when:**
- Ansible playbook and inventory are created.
- FastAPI endpoint can trigger Ansible backup.
- Backup functionality is tested and working.

### Step 5 – Create Your Dockerfile
**What you're doing:** Defining how to build your custom container image.

Create `Dockerfile`:
```dockerfile
# Use official Python runtime as base image
FROM python:3.11-slim

# Set working directory in container
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    openssh-client \
    sshpass \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first (for better caching)
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app/ ./app/
COPY ansible/ ./ansible/

# Create necessary directories
RUN mkdir -p logs data backups

# Expose port for FastAPI
EXPOSE 8000

# Set environment variables
ENV PYTHONPATH=/app
ENV ANSIBLE_HOST_KEY_CHECKING=False

# Command to run the application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Create `.dockerignore`:
```
__pycache__
*.pyc
*.pyo
*.pyd
.git
.gitignore
README.md
.env
.venv
Dockerfile
.dockerignore
```

**LOG REQUIREMENT:** `DOCKERFILE_CREATED ts=<timestamp> base_image=python:3.11-slim`

**Done when:**
- `Dockerfile` is created with proper structure and dependencies.
- `.dockerignore` excludes unnecessary files from build context.
- Dockerfile follows best practices (specific base image, proper layer ordering, non-root user considerations).

### Step 6 – Build Your Custom Container Image
**What you're doing:** Using Docker to build your containerized application.

1. **Build the Image**
```bash
docker build -t network-automation-api:v1.0 .
```

2. **Verify the Build**
```bash
docker images | grep network-automation-api
docker history network-automation-api:v1.0
```

3. **Inspect the Image**
```bash
docker inspect network-automation-api:v1.0
```

**LOG REQUIREMENT:** `IMAGE_BUILD_OK ts=<timestamp> image=network-automation-api:v1.0 size=<size_mb>MB`

**Save build output:**
```bash
docker build -t network-automation-api:v1.0 . > logs/build_output.log 2>&1
```

**Done when:**
- Docker image builds successfully without errors.
- Image appears in `docker images` output.
- Build output is saved to logs.
- Log entry records successful build with image details.

### Step 7 – Run and Test Your Container
**What you're doing:** Launching your containerized API and validating all endpoints.

1. **Run the Container**
```bash
docker run -d -p 8000:8000 --name network-api network-automation-api:v1.0
```

2. **Verify Container is Running**
```bash
docker ps
docker logs network-api
```

3. **Test All Endpoints**
```bash
# Health check
curl http://localhost:8000/

# ASCII Art
curl http://localhost:8000/ascii/Docker

# Dad Joke
curl http://localhost:8000/joke

# Draw Card
curl http://localhost:8000/card

# Trigger Backup
curl -X POST http://localhost:8000/backup
```

4. **Save API Responses**
```bash
curl http://localhost:8000/ > data/api_responses/health.json
curl http://localhost:8000/ascii/Automation > data/api_responses/ascii.json
curl http://localhost:8000/joke > data/api_responses/joke.json
curl http://localhost:8000/card > data/api_responses/card.json
curl -X POST http://localhost:8000/backup > data/api_responses/backup.json
```

**LOG REQUIREMENT:** `CONTAINER_RUN_OK ts=<timestamp> container_id=<container_id> port=8000`
**LOG REQUIREMENT:** `API_TEST_OK ts=<timestamp> endpoint=/ status=200`
**LOG REQUIREMENT:** `API_TEST_OK ts=<timestamp> endpoint=/ascii/Docker status=200`
**LOG REQUIREMENT:** `API_TEST_OK ts=<timestamp> endpoint=/joke status=200`
**LOG REQUIREMENT:** `API_TEST_OK ts=<timestamp> endpoint=/card status=200`
**LOG REQUIREMENT:** `API_TEST_OK ts=<timestamp> endpoint=/backup status=200`

**Done when:**
- Container runs successfully in detached mode.
- All API endpoints respond correctly.
- API responses are saved to data directory.
- Container logs show no errors.
- All endpoint tests are logged.

### Step 8 – Create Documentation and Version Management
**What you're doing:** Documenting your container and demonstrating Docker version management.

1. **Create Usage Documentation**
Create `docs/container_usage.md`:
```markdown
# Network Automation API Container

## Building the Container
\`\`\`bash
docker build -t network-automation-api:v1.0 .
\`\`\`

## Running the Container
\`\`\`bash
docker run -d -p 8000:8000 --name network-api network-automation-api:v1.0
\`\`\`

## Stopping and Cleaning Up
\`\`\`bash
docker stop network-api
docker rm network-api
\`\`\`
```

2. **Create API Documentation**
Create `docs/api_endpoints.md` with all endpoint documentation and cURL examples.

3. **Demonstrate Versioning**
```bash
# Tag current version
docker tag network-automation-api:v1.0 network-automation-api:latest

# Show all tags
docker images network-automation-api
```

4. **Save Container Logs**
```bash
docker logs network-api > data/container_logs.txt
```

**LOG REQUIREMENT:** `DOCUMENTATION_CREATED ts=<timestamp> files=2`
**LOG REQUIREMENT:** `VERSION_TAGGED ts=<timestamp> versions=v1.0,latest`

**Done when:**
- Documentation files are created and comprehensive.
- Container is properly tagged with version information.
- Container logs are saved to data directory.

### Step 9 – Advanced Container Operations
**What you're doing:** Demonstrating container lifecycle management and troubleshooting skills.

1. **Container Inspection**
```bash
# View container details
docker inspect network-api

# Check resource usage
docker stats network-api --no-stream

# Access container shell (if needed)
docker exec -it network-api /bin/bash
```

2. **Container Management**
```bash
# Stop container gracefully
docker stop network-api

# Start stopped container
docker start network-api

# Remove container and image
docker rm network-api
docker rmi network-automation-api:v1.0
```

3. **Rebuild and Test Cycle**
- Make a small change to your API
- Rebuild with v1.1 tag
- Run new version and compare

**LOG REQUIREMENT:** `CONTAINER_LIFECYCLE_OK ts=<timestamp> operations=inspect,stop,start,remove`

**Done when:**
- Container inspection commands execute successfully.
- Container can be stopped, started, and removed cleanly.
- Advanced operations are logged.

### Step 10 – Commit, Push, and Final Validation
**What you're doing:** Submitting your completed containerized application.

1. **Final Testing**
- Ensure all endpoints work correctly
- Verify all logs contain required entries
- Check that documentation is complete

2. **Commit and Push**
```bash
git add .
git commit -m "Lab 8 complete: Docker containerized FastAPI with network automation"
git push
```

3. **Final Log Entry**
**LOG REQUIREMENT:** `LAB8_END ts=<timestamp> container_built=true api_tested=true documentation_complete=true`

**Done when:**
- All deliverables are present in the repository.
- GitHub Actions (if configured) shows successful validation.
- Final log entry confirms lab completion.

---

## :wrench: Troubleshooting & Common Pitfalls

**1. Docker Not Running**
- **Symptom:** `Cannot connect to the Docker daemon`
- **Fix:** Ensure Docker Desktop is running, or start Docker daemon on Linux: `sudo systemctl start docker`

**2. Port Already in Use**
- **Symptom:** `Port 8000 is already allocated`
- **Fix:** Use a different port: `docker run -d -p 8001:8000 --name network-api network-automation-api:v1.0`

**3. Container Build Fails**
- **Symptom:** Docker build exits with error during `pip install`
- **Fix:** Check `requirements.txt` for typos, ensure base image is accessible, clear Docker cache: `docker system prune`

**4. API Endpoints Return 500 Errors**
- **Symptom:** cURL requests return internal server errors
- **Fix:** Check container logs: `docker logs network-api`. Common issues: missing environment variables, import errors, or external API connectivity.

**5. Ansible Integration Fails**
- **Symptom:** `/backup` endpoint returns errors about SSH or authentication
- **Fix:** Verify DevNet sandbox credentials, check inventory file format, ensure container has network access.

**6. Image Size Too Large**
- **Symptom:** Docker image is several GB in size
- **Fix:** Use `.dockerignore`, choose slim base images, combine RUN commands, remove unnecessary packages.

**7. Container Stops Immediately**
- **Symptom:** `docker ps` shows no running containers after `docker run`
- **Fix:** Check logs: `docker logs <container_name>`. Often caused by application startup errors or missing files.

**8. Permission Denied Errors**
- **Symptom:** Container can't write files or access directories
- **Fix:** Ensure directories exist in Dockerfile, consider user permissions, check volume mounts if using them.

---

## :bulb: Pro Tips

**Optimize Docker Builds**
- Order Dockerfile commands from least to most frequently changing
- Use specific version tags for base images
- Leverage multi-stage builds for production images

**Container Security Best Practices**
- Don't run containers as root user
- Use official base images when possible
- Keep images updated and scan for vulnerabilities

**Development Workflow**
- Use Docker Compose for multi-container applications
- Mount code volumes during development for faster iteration
- Use container registries (Docker Hub, AWS ECR) for sharing images

**Debugging Containers**
- Use `docker exec -it <container> /bin/bash` to inspect running containers
- Add health checks to your Dockerfile
- Use structured logging in your applications

**Performance Considerations**
- Use `.dockerignore` to reduce build context size
- Consider using Alpine Linux for smaller images
- Monitor container resource usage with `docker stats`

---

## Grading and Points Breakdown

> **NOTE: ZERO CREDIT CONDITIONS**
- 0 PTS FOR THE LAB IF:
    - Docker container doesn't build or run successfully
    - Hard-coded credentials are found in any files
    - Required API endpoints don't function properly

| Step                    | Requirement                                           | Points |
|------------------------|------------------------------------------------------|--------|
| 1. Docker Setup        | Docker installation verified and hello-world runs    | 5      |
| 2. Hello World         | Successfully run and log hello-world container       | 5      |
| 3. FastAPI App         | Create functional FastAPI with required endpoints    | 15     |
| 4. Ansible Integration | Working network backup endpoint using Ansible        | 15     |
| 5. Dockerfile          | Proper Dockerfile with best practices                | 10     |
| 6. Container Build     | Successfully build custom image                       | 10     |
| 7. Container Run       | Run container and test all endpoints with cURL       | 20     |
| 8. Documentation       | Complete usage and API documentation                  | 10     |
| 9. Advanced Ops        | Demonstrate container lifecycle management            | 5      |
| 10. Logging & Submission | All required log entries and successful push       | 5      |

**Total: 100 Points**

> **Bonus Points (5 pts):** Implement additional endpoints beyond requirements or demonstrate advanced Docker features (multi-stage builds, health checks, etc.)

---

## Submission Checklist
- [ ] Docker hello-world container runs successfully
- [ ] Custom FastAPI application with all required endpoints
- [ ] Dockerfile builds image without errors
- [ ] Container runs and serves API on port 8000
- [ ] All endpoints tested with cURL and responses saved
- [ ] Ansible integration works for device backup
- [ ] Complete documentation in docs/ directory
- [ ] All required log entries present
- [ ] Code committed and pushed to GitHub
- [ ] Container can be stopped, started, and removed cleanly

:green_checkmark: **Final validation: Your containerized API should be accessible at `http://localhost:8000` and respond to all documented endpoints.**