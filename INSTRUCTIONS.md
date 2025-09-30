# Instructions — Lab 8 — Docker Containers for Network Engineers

> **Before you begin:** This lab runs **natively on your host** (not inside a dev container). Ensure Docker is installed and the daemon is running. Create `logs/`, `data/api_responses/`, and `docs/` if they don’t exist.


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


## Submission Checklist
- [ ] hello-world runs and is logged.
- [ ] FastAPI app exposes /, /ascii/{text}, /joke, /card, and /backup.
- [ ] Dockerfile builds image without errors; build logs saved.
- [ ] Container runs on port 8000; endpoint responses saved under `data/api_responses/`.
- [ ] Ansible backup endpoint functions; inventory/playbook present.
- [ ] Docs created under `docs/`.
- [ ] All required markers present in `logs/lab8_docker.log`.
