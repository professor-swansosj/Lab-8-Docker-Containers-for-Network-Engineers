# Lab 8 — Docker Containers for Network Engineers

**Course:** Software Defined Networking  
**Module:** Network Automation Fundamentals • **Lab #:** 8  
**Estimated Time:** 120–150 minutes

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


## Resources
- [Docker Documentation](https://docs.docker.com/)- [FastAPI](https://fastapi.tiangolo.com/)- [Uvicorn](https://www.uvicorn.org/)- [Requests (Python)](https://requests.readthedocs.io/en/latest/)- [Ansible](https://docs.ansible.com/)- [pyfiglet](https://pypi.org/project/pyfiglet/)- [Dad Jokes API](https://icanhazdadjoke.com/api) — Use Accept: application/json- [Deck of Cards API](https://deckofcardsapi.com/)- [Cisco DevNet Sandboxes](https://developer.cisco.com/site/sandbox/)

## FAQ
**Q:** Docker says it cannot connect to the daemon.  
**A:** Start Docker Desktop (or `sudo systemctl start docker` on Linux) and re-run.

**Q:** Container runs but endpoints 500.  
**A:** Check container logs (`docker logs <name>`), confirm imports and network access.

**Q:** Backup endpoint fails.  
**A:** Verify inventory credentials and connectivity to DevNet sandbox; ensure Ansible is installed in the image.



## Deliverables
- Standard README explaining Docker basics, goals, grading, and tips.
- Stepwise INSTRUCTIONS covering hello-world, FastAPI, Dockerfile, build/run/test, Ansible integration, docs, and lifecycle.
- Grading: **100 points**

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



## Autograder Notes
- Log file: `logs/*.log`
- Required markers: `LAB8_START`, `DOCKER_HELLO_OK`, `FASTAPI_APP_CREATED`, `ANSIBLE_INTEGRATION_OK`, `DOCKERFILE_CREATED`, `IMAGE_BUILD_OK`, `CONTAINER_RUN_OK`, `API_TEST_OK`, `DOCUMENTATION_CREATED`, `VERSION_TAGGED`, `CONTAINER_LIFECYCLE_OK`, `LAB8_END`

## License
© 2025 Your Name — Classroom use.

# HAPPY CODING!