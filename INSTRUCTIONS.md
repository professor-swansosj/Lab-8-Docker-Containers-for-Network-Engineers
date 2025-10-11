# Instructions — Lab 8 — Docker Containers for Network Engineers

## Objectives
- Author a Dockerfile to containerize a FastAPI webhook service.
- Add a new FastAPI path that triggers an Ansible backup playbook.
- Package required dependencies (FastAPI, Uvicorn, requests, Netmiko, ncclient, Ansible, collections).
- Use environment variables and volumes for credentials and persistent artifacts.
- Build, run, test, and log deterministic markers for autograding.

## Prerequisites
- Python 3.11 (via the provided dev container)
- Accounts: GitHub, Cisco DevNet
- Devices/Sandboxes: Local Docker Engine, Cisco DevNet Always-On Catalyst 8k/9k (for backup target)
- Technical: - FastAPI app from Lab 6 (webhooks working).
- Basic Docker commands: build, run, logs, stop, rm.
- Ansible basics (inventory, playbooks, collections).
- GitHub Classroom workflow (clone, commit, push, PR).

## Overview
You will containerize your FastAPI webhook service from Lab 6 and extend it with a new endpoint that, when called, runs an Ansible playbook to back up device configurations from the Catalyst 8k and 9k sandboxes. You will write a production-minded Dockerfile, install Python and Ansible dependencies, prepare an inventory and playbook in the image, and expose port 8000. Runtime environment variables provide credentials; a bind-mounted volume preserves backups and logs. You will build, run, and test the container, saving artifacts and log markers for autograding.


> **Before you begin:** Verify Docker is installed and the Docker daemon is running (`docker version` works). Ensure you can write to `data/` and `logs/`. Have the IP/hostnames for the Cat8k and Cat9k sandboxes ready.


## Resources
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)- [FastAPI](https://fastapi.tiangolo.com/)- [Uvicorn](https://www.uvicorn.org/)- [Ansible docs](https://docs.ansible.com/)- [cisco.ios collection](https://docs.ansible.com/ansible/latest/collections/cisco/ios/)
## Deliverables
- `Dockerfile` that builds a runnable image for the FastAPI app.
- `src/app.py` FastAPI webhook with a new POST path `/backup/ansible`.
- Ansible artifacts inside repo: `playbooks/backup.yml`, `inventories/inventory.yml`, `requirements.yml`.
- Build and run transcripts saved to `data/docker_build.txt` and `data/docker_run.txt`.
- Backups saved under `data/backups/` (e.g., `cat8k_running_config.txt`, `cat9k_running_config.txt`).
- `logs/lab8.log` with required markers.
- Pull request open to `main` with all artifacts committed.
- Grading: **75 points**

Follow these steps in order.

> **Logging Requirement:** Write progress to `logs/lab8.log` as you complete each step.

## Step 1 — Clone the Repository
**Goal:** Get your starter locally.

**What to do:**  
Clone your GitHub Classroom repo and `cd` into it. Create `src/`, `playbooks/`, `inventories/`,
`data/backups/`, and `logs/` if missing.
Initialize the lab log: `echo 'LAB8_START' >> logs/lab8.log`


**You're done when:**  
- Folders exist and `LAB8_START` appears in `logs/lab8.log`.


**Log marker to add:**  
`[LAB8_START]`

## Step 2 — Dev Container Check (Optional)
**Goal:** Confirm Python app still runs locally.

**What to do:**  
If using dev containers, reopen now and verify `python -c "import fastapi, uvicorn; print('OK')"` prints OK.
Append `[STEP 2] Dev Container Started` to `logs/lab8.log` if you used it.


**You're done when:**  
- Local run sanity-checked OR dev container confirmed.


**Log marker to add:**  
`[[STEP 2] Dev Container Started]`

## Step 3 — Extend FastAPI App
**Goal:** Add `/backup/ansible` POST endpoint.

**What to do:**  
In `src/app.py` (from Lab 6), add a new POST handler:
  - Validates token (same `X-Webhook-Token` approach).
  - Invokes an Ansible playbook (subprocess) to back up Cat8k and Cat9k running-configs.
  - Writes backups to `data/backups/` and returns a JSON summary (hostnames + file paths).
  - Logs `BACKUP_CALLED` and `BACKUP_FILES_SAVED`.
Keep existing endpoints from Lab 6 working.


**You're done when:**  
- New endpoint implemented without breaking other paths.


**Log marker to add:**  
`[WEBHOOKS_OK]`

## Step 4 — Author the Ansible Playbook & Inventory
**Goal:** Create portable IaC inside the container.

**What to do:**  
Create:
  - `requirements.yml` (include `cisco.ios` and `ansible.netcommon`).
  - `inventories/inventory.yml` with hosts `cat8k` and `cat9k` (use env vars for creds; no secrets in repo).
  - `playbooks/backup.yml` that:
      * Runs against group `iosxe`
      * Uses `cisco.ios.ios_command` (e.g., `show running-config`) or `cisco.ios.ios_config` with `backup: true`
      * Registers output and writes files to `/app/data/backups/` (container path)
Log `ANSIBLE_ASSETS_OK`.


**You're done when:**  
- Inventory, requirements, and playbook validate (`ansible-playbook --syntax-check playbooks/backup.yml`).


**Log marker to add:**  
`[ANSIBLE_ASSETS_OK]`

## Step 5 — Write the Dockerfile
**Goal:** Containerize app + Ansible cleanly.

**What to do:**  
Create `Dockerfile` that:
  - Uses a slim Python 3.11 base.
  - Installs system deps needed by Ansible/SSH (e.g., `openssh-client`, `sshpass`, `git`).
  - `pip install` Python deps (fastapi, uvicorn, requests, netmiko, ncclient, xmltodict, ansible, ansible-pylibssh).
  - Copies `src/`, `playbooks/`, `inventories/`, `requirements.yml`, and `logs/` scaffold into `/app`.
  - Runs `ansible-galaxy collection install -r /app/requirements.yml` at build time.
  - Creates non-root user; sets `WORKDIR /app`.
  - Exposes `8000` and adds a `HEALTHCHECK` hitting `/health` endpoint.
  - `CMD ["uvicorn", "src.app:app", "--host", "0.0.0.0", "--port", "8000"]`
Log `DOCKERFILE_OK`.


**You're done when:**  
- Dockerfile passes `hadolint` (if available) and looks production-minded.


**Log marker to add:**  
`[DOCKERFILE_OK]`

## Step 6 — Build the Image
**Goal:** Produce a tagged image.

**What to do:**  
Run: `docker build -t sdn-fastapi-backup:lab8 . | tee data/docker_build.txt`
On success, append `BUILD_OK` to `logs/lab8.log`.


**You're done when:**  
- Image `sdn-fastapi-backup:lab8` exists; build transcript saved.


**Log marker to add:**  
`[BUILD_OK]`

## Step 7 — Run the Container
**Goal:** Start service with env + volume.

**What to do:**  
Run (example):
  docker run -d --name sdn-lab8 \
    -p 8000:8000 \
    -e WEBHOOK_TOKEN=<your_token> \
    -e ANSIBLE_USER=<sandbox_user> \
    -e ANSIBLE_PASSWORD=<sandbox_pass> \
    -e CAT8K_HOST=<ip_or_fqdn> \
    -e CAT9K_HOST=<ip_or_fqdn> \
    -v "$PWD/data:/app/data" \
    sdn-fastapi-backup:lab8 | tee data/docker_run.txt
Append `RUN_OK` to `logs/lab8.log`.


**You're done when:**  
- Container is running and mapped to localhost:8000.


**Log marker to add:**  
`[RUN_OK]`

## Step 8 — Health & Endpoint Tests
**Goal:** Verify service is alive and endpoints respond.

**What to do:**  
Hit health (if implemented) and a simple endpoint (e.g., `/joke`) with the token:
  curl -s -X POST http://localhost:8000/joke -H "X-Webhook-Token: <your_token>"
Save brief test notes to `data/test_notes.txt`. Append `HEALTH_OK` to log.


**You're done when:**  
- At least one non-backup endpoint returns 200.


**Log marker to add:**  
`[HEALTH_OK]`

## Step 9 — Trigger Ansible Backup
**Goal:** Execute the new endpoint and persist artifacts.

**What to do:**  
Call:
  curl -s -X POST http://localhost:8000/backup/ansible \
    -H "X-Webhook-Token: <your_token>" \
    -H "Content-Type: application/json" \
    -d '{"targets":["cat8k","cat9k"]}'
Ensure output files appear in `data/backups/` and JSON response lists file paths.
Append `BACKUP_OK` and `BACKUP_FILES_SAVED` to the log.


**You're done when:**  
- `data/backups/` contains both device backups with non-empty content.


**Log marker to add:**  
`[BACKUP_OK, BACKUP_FILES_SAVED]`

## Step 10 — Finalize & Submit
**Goal:** Stop, clean, and open PR.

**What to do:**  
Append `LAB8_END` to `logs/lab8.log`. Optionally stop/remove the container:
  docker stop sdn-lab8 && docker rm sdn-lab8
Commit and push all artifacts; open a PR targeting `main`.


**You're done when:**  
- PR open; all required files present.


**Log marker to add:**  
`[LAB8_END]`


## FAQ
**Q:** Backups didn't appear under data/backups/  
**A:** Check your `-v $PWD/data:/app/data` bind mount and ensure the playbook writes to `/app/data/backups/`.

**Q:** 401 Unauthorized on POST  
**A:** Send the header `X-Webhook-Token: <token>`; ensure the container has `WEBHOOK_TOKEN` set.

**Q:** Ansible authentication fails  
**A:** Provide `ANSIBLE_USER` / `ANSIBLE_PASSWORD` env vars and correct `CAT8K_HOST` / `CAT9K_HOST`.

**Q:** ncclient/Netmiko missing  
**A:** Confirm they're installed in the image; rebuild after editing the Dockerfile.


## 🔧 Troubleshooting & Pro Tips
**Keep secrets out of Git**  
*Symptom:* Credentials leaked in repo.  
*Fix:* Use env vars or a `.env` loaded by your run script; never commit secrets.

**Repeatable images**  
*Symptom:* Builds differ across machines.  
*Fix:* Pin Python base image tag and key pip package versions where possible.

**Faster builds**  
*Symptom:* Slow rebuilds after code edits.  
*Fix:* Copy dependency files earlier in the Dockerfile to leverage layer caching.


## Grading Breakdown
| Step | Requirement | Points |
|---|---|---|
| FastAPI | Existing endpoints retained; new `/backup/ansible` implemented (`WEBHOOKS_OK`) | 10 |
| Ansible | Inventory + playbook + requirements set up (`ANSIBLE_ASSETS_OK`) | 10 |
| Dockerfile | Clean, production-minded Dockerfile (`DOCKERFILE_OK`) | 5 |
| Build | Image builds; transcript saved (`BUILD_OK` + `data/docker_build.txt`) | 10 |
| Run | Container runs with env + volume; transcript saved (`RUN_OK` + `data/docker_run.txt`) | 5 |
| Health | Service reachable; non-backup endpoint works (`HEALTH_OK`) | 5 |
| Backup | Backup endpoint succeeds; files present (`BACKUP_OK`, `BACKUP_FILES_SAVED`) | 15 |
| Submission | PR open; log hygiene (`LAB8_START`/`LAB8_END` present) | 15 |
| **Total** |  | **75** |

## Autograder Notes
- Log file: `logs/lab8.log`
- Required markers: `LAB8_START`, `[STEP 2] Dev Container Started`, `WEBHOOKS_OK`, `ANSIBLE_ASSETS_OK`, `DOCKERFILE_OK`, `BUILD_OK`, `RUN_OK`, `HEALTH_OK`, `BACKUP_OK`, `BACKUP_FILES_SAVED`, `LAB8_END`

## Submission Checklist
- [ ] `Dockerfile` builds `sdn-fastapi-backup:lab8` successfully.
- [ ] Container runs exposing :8000 with token + device env vars and a `data/` volume.
- [ ] `/backup/ansible` returns 200 and creates backups under `data/backups/`.
- [ ] Transcripts saved to `data/docker_build.txt` and `data/docker_run.txt`.
- [ ] `logs/lab8.log` includes all required markers.
- [ ] Pull request open to `main` before deadline.
