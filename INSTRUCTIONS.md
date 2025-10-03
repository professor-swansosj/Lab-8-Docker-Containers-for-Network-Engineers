# Instructions — Lab 8 — Docker Containers for Network Engineers

## Objectives
- Run a first Docker container (hello-world) to validate the local Docker environment.
- Explain the difference between Docker images and containers.
- Create a Dockerfile and build a custom image for a FastAPI app.
- Containerize a FastAPI application with multiple endpoints (ASCII art, external APIs, backup).
- Integrate Ansible inside the container to back up a network device.
- Test containerized endpoints with cURL and save responses.
- Tag, run, inspect, and manage container lifecycle operations.
- Produce logs and artifacts for autograding.

## Prerequisites
- Python 3.11 (via the provided dev container)
- Accounts: GitHub
- Devices/Sandboxes: Local machine with Docker (Desktop/Engine), Cisco DevNet Always-On Sandbox (for backup endpoint)

## Overview
You’ll shift from *using* dev containers to actually *building* and running your own Docker images. First validate Docker with hello-world, then package a FastAPI app that exposes endpoints for ASCII art, calling public APIs, and triggering a network backup via Ansible. You’ll test with cURL, save artifacts, tag images, and exercise container lifecycle commands end-to-end.


> **Before you begin:** This lab runs **natively on your host** (not inside a dev container). Ensure Docker is installed and the daemon is running. Create `logs/`, `data/api_responses/`, and `docs/` if they don’t exist.


## Resources
- [Docker Documentation](https://docs.docker.com/)- [FastAPI](https://fastapi.tiangolo.com/)- [Uvicorn](https://www.uvicorn.org/)- [Requests (Python)](https://requests.readthedocs.io/en/latest/)- [Ansible](https://docs.ansible.com/)- [pyfiglet](https://pypi.org/project/pyfiglet/)- [Dad Jokes API](https://icanhazdadjoke.com/api) — Use Accept: application/json- [Deck of Cards API](https://deckofcardsapi.com/)- [Cisco DevNet Sandboxes](https://developer.cisco.com/site/sandbox/)
## Deliverables
- Standard README explaining Docker basics, goals, grading, and tips.
- Stepwise INSTRUCTIONS covering hello-world, FastAPI, Dockerfile, build/run/test, Ansible integration, docs, and lifecycle.
- Grading: **100 points**

Follow these steps in order.

> **Logging Requirement:** Write progress to `logs/*.log` as you complete each step.

## Step 1 — Setup local Docker
**Goal:** Verify Docker is ready and initialize repo structure.

**What to do:**  
```bash
docker --version
docker info
git clone <repo-url> && cd <repo-name>
mkdir -p app ansible data/api_responses docs logs
```
Add this logging helper at the top of `app/main.py`:
```python
from datetime import datetime, timezone
import os
def now_iso(): return datetime.now(timezone.utc).isoformat().replace("+00:00","Z")
def log(line, path):
    os.makedirs(os.path.dirname(path), exist_ok=True)
    with open(path,"a",encoding="utf-8") as f: f.write(f"{line}\n")
# example: log(f"LAB8_START ts={now_iso()}", "logs/lab8_docker.log")
```


**You’re done when:**  
- `docker --version` and `docker info` succeed.
- `logs/lab8_docker.log` contains a start timestamp.


**Log marker to add:**  
`[LAB8_START]`

## Step 2 — Run hello-world
**Goal:** Validate pull/run and local cache.

**What to do:**  
```bash
docker run hello-world
docker ps -a
docker images
```


**You’re done when:**  
- Terminal prints "Hello from Docker!"
- Image and container are visible via CLI.


**Log marker to add:**  
`[DOCKER_HELLO_OK]`

## Step 3 — Build the FastAPI app
**Goal:** Create endpoints and supporting modules.

**What to do:**  
Create `requirements.txt`:
```text
fastapi==0.104.1
uvicorn[standard]==0.24.0
requests==2.31.0
pyfiglet==1.0.2
ansible==8.5.0
```
Create `app/main.py` with at least:
- `GET /` (health),
- `GET /ascii/{text}`,
- `GET /joke`,
- `GET /card`,
- `POST /backup` (calls Ansible).
Add `app/reverse_api.py` and `app/ansible_backup.py` as helpers.
Test locally (optional): `uvicorn app.main:app --reload`.


**You’re done when:**  
Local run works; endpoints are defined.

**Log marker to add:**  
`[FASTAPI_APP_CREATED]`

## Step 4 — Integrate Ansible backup
**Goal:** Add device backup via Ansible playbook.

**What to do:**  
Create `ansible/backup-playbook.yml` and `ansible/inventory.yml` for the DevNet sandbox.
In `app/ansible_backup.py`, run `ansible-playbook` as a subprocess from `/app`.


**You’re done when:**  
Backup endpoint triggers playbook and reports status.

**Log marker to add:**  
`[ANSIBLE_INTEGRATION_OK]`

## Step 5 — Write Dockerfile and dockerignore
**Goal:** Define image build.

**What to do:**  
Create `Dockerfile` (python:3.11-slim, install deps, copy app/ and ansible/, create dirs, expose 8000, run uvicorn).
Create `.dockerignore` to keep image lean.


**You’re done when:**  
Files exist and are syntactically valid.

**Log marker to add:**  
`[DOCKERFILE_CREATED]`

## Step 6 — Build the image
**Goal:** Produce a tagged image and save build output.

**What to do:**  
```bash
docker build -t network-automation-api:v1.0 . > logs/build_output.log 2>&1
docker images | grep network-automation-api
docker history network-automation-api:v1.0
docker inspect network-automation-api:v1.0 | wc -l
```


**You’re done when:**  
Image builds successfully; logs captured.

**Log marker to add:**  
`[IMAGE_BUILD_OK]`

## Step 7 — Run and test
**Goal:** Launch container and verify endpoints.

**What to do:**  
```bash
docker run -d -p 8000:8000 --name network-api network-automation-api:v1.0
docker ps
docker logs network-api > data/container_logs.txt
# cURL tests
curl http://localhost:8000/ > data/api_responses/health.json
curl http://localhost:8000/ascii/Automation > data/api_responses/ascii.json
curl http://localhost:8000/joke > data/api_responses/joke.json
curl http://localhost:8000/card > data/api_responses/card.json
curl -X POST http://localhost:8000/backup > data/api_responses/backup.json
```


**You’re done when:**  
All endpoints return 200 and responses are saved.

**Log marker to add:**  
`[CONTAINER_RUN_OK, API_TEST_OK]`

## Step 8 — Docs and versioning
**Goal:** Document usage and tag versions.

**What to do:**  
Create `docs/container_usage.md` and `docs/api_endpoints.md` with build/run and cURL examples.
Tag the image:
```bash
docker tag network-automation-api:v1.0 network-automation-api:latest
docker images network-automation-api
```


**You’re done when:**  
Docs exist; image shows v1.0 and latest.

**Log marker to add:**  
`[DOCUMENTATION_CREATED, VERSION_TAGGED]`

## Step 9 — Lifecycle & troubleshooting
**Goal:** Operate container confidently.

**What to do:**  
```bash
docker inspect network-api | head -n 20
docker stats network-api --no-stream
docker stop network-api && docker start network-api
docker rm -f network-api
docker rmi network-automation-api:v1.0
```


**You’re done when:**  
Inspect/stop/start/remove succeed without errors.

**Log marker to add:**  
`[CONTAINER_LIFECYCLE_OK]`

## Step 10 — Commit, push, finalize
**Goal:** Submit with complete artifacts and logs.

**What to do:**  
Ensure logs include all required markers; commit and push:
```bash
git add .
git commit -m "Lab 8 complete: Dockerized FastAPI + Ansible backup"
git push
```
Add a final log line.


**You’re done when:**  
PR opens and Verify Docs is green.

**Log marker to add:**  
`[LAB8_END]`


## FAQ
**Q:** Docker says it cannot connect to the daemon.  
**A:** Start Docker Desktop (or `sudo systemctl start docker` on Linux) and re-run.

**Q:** Container runs but endpoints 500.  
**A:** Check container logs (`docker logs <name>`), confirm imports and network access.

**Q:** Backup endpoint fails.  
**A:** Verify inventory credentials and connectivity to DevNet sandbox; ensure Ansible is installed in the image.


## 🔧 Troubleshooting & Pro Tips
**Docker not running**  
*Symptom:* Cannot connect to Docker daemon  
*Fix:* Start Docker Desktop or `sudo systemctl start docker` (Linux).

**Port already in use**  
*Symptom:* Port 8000 is allocated  
*Fix:* Use `-p 8001:8000` or free the port.

**Large image size**  
*Symptom:* Image is several GB  
*Fix:* Use slim base images, `.dockerignore`, and pin packages.

**Permissions in container**  
*Symptom:* Cannot write logs  
*Fix:* Ensure directories exist and consider user permissions; create `logs/` in image.


## Grading Breakdown
| Step | Requirement | Points |
|---|---|---|
| 1 | Docker installation verified | 5 |
| 2 | hello-world container runs and is logged | 5 |
| 3 | FastAPI app created with required endpoints | 15 |
| 4 | Ansible backup integration working | 15 |
| 5 | Dockerfile follows best practices | 10 |
| 6 | Image build completes; output saved | 10 |
| 7 | Container runs; all endpoints tested via cURL | 20 |
| 8 | Documentation created (usage + API) | 10 |
| 9 | Lifecycle ops demonstrated (inspect/stop/start/remove) | 5 |
| 10 | All required logs and final submission | 5 |
| **Total** |  | **100** |

## Autograder Notes
- Log file: `logs/*.log`
- Required markers: `LAB8_START`, `DOCKER_HELLO_OK`, `FASTAPI_APP_CREATED`, `ANSIBLE_INTEGRATION_OK`, `DOCKERFILE_CREATED`, `IMAGE_BUILD_OK`, `CONTAINER_RUN_OK`, `API_TEST_OK`, `DOCUMENTATION_CREATED`, `VERSION_TAGGED`, `CONTAINER_LIFECYCLE_OK`, `LAB8_END`

## Submission Checklist
- [ ] hello-world runs and is logged.
- [ ] FastAPI app exposes /, /ascii/{text}, /joke, /card, and /backup.
- [ ] Dockerfile builds image without errors; build logs saved.
- [ ] Container runs on port 8000; endpoint responses saved under `data/api_responses/`.
- [ ] Ansible backup endpoint functions; inventory/playbook present.
- [ ] Docs created under `docs/`.
- [ ] All required markers present in `logs/lab8_docker.log`.
