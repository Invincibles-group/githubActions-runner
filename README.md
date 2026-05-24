# GitHub Actions Runner - DevSecOps Pipeline

A Python Flask web application with a fully automated **DevSecOps CI/CD pipeline** built using GitHub Actions reusable workflows.

## Application

A simple Flask web app served via Gunicorn, containerized with Docker.

| Route | Description |
|-------|-------------|
| `/` | Renders `index.html` |
| `/health` | Returns health check response |

**Tech Stack:** Python 3.13, Flask, Gunicorn, Docker

## Pipeline Architecture

The pipeline is triggered on every push to `main` and executes the following stages in order:

```
┌─────────────────────────────────────────────────────┐
│              CI - Security Checks (parallel)         │
├──────────────┬──────────────┬───────────┬───────────┤
│ Code Quality │ Secret Scan  │ Dep Scan  │Docker Lint│
└──────┬───────┴──────┬───────┴─────┬─────┴─────┬─────┘
       └──────────────┴─────────────┴───────────┘
                          │
                    ┌─────▼─────┐
                    │   Build   │
                    │ & Push    │
                    └─────┬─────┘
                          │
                   ┌──────▼──────┐
                   │ Image Scan  │
                   │  (Trivy)    │
                   └──────┬──────┘
                          │
                    ┌─────▼─────┐
                    │  Deploy   │
                    └───────────┘
```

### Stage Details

| Stage | Workflow | Tool | Purpose |
|-------|---------|------|---------|
| Code Quality | `code-quality.yml` | flake8, bandit | Linting and static security analysis across Python 3.11–3.13 |
| Secret Scan | `secrets-scan.yml` | gitleaks | Detect hardcoded secrets and API keys |
| Dependency Scan | `dependency-scan.yml` | pip-audit | Audit Python dependencies for known CVEs |
| Docker Lint | `docker-lint.yml` | hadolint | Validate Dockerfile best practices |
| Build & Push | `docker-build-push.yml` | docker/build-push-action | Build image and push to Docker Hub |
| Image Scan | `image-scan.yml` | Trivy | Scan container image for HIGH/CRITICAL vulnerabilities |
| Deploy | `deploy-to-server.yml` | appleboy/ssh-action, scp-action | Deploy to EC2 via SSH using Docker Compose |

## Repository Structure

```
.
├── app.py                  # Flask application
├── templates/
│   └── index.html          # HTML template
├── requirements.txt        # Python dependencies
├── Dockerfile              # Container image definition
├── docker-compose.yml      # Compose config for deployment
├── .trivyignore            # Trivy CVE suppressions
└── .github/workflows/
    ├── devsecops-pipeline.yml   # Orchestrator (main pipeline)
    ├── code-quality.yml         # Linting & static analysis
    ├── secrets-scan.yml         # Secret detection
    ├── dependency-scan.yml      # Dependency audit
    ├── docker-lint.yml          # Dockerfile linting
    ├── docker-build-push.yml    # Docker build & push
    ├── image-scan.yml           # Container image scanning
    └── deploy-to-server.yml     # SSH deploy to server
```

## Required Secrets & Variables

Configure these in **Settings → Secrets and variables → Actions**:

### Secrets
| Name | Description |
|------|-------------|
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `EC2_SSH_HOST` | Server IP/hostname |
| `EC2_SSH_USER` | SSH username |
| `EC2_PRIVATE_KEY` | SSH private key (PEM) |

### Variables
| Name | Description |
|------|-------------|
| `DOCKERHUB_USER` | Docker Hub username |

## Running Locally

```bash
# Install dependencies
pip install -r requirements.txt

# Run the app
flask run --port 8082

# Or with Docker
docker compose up --build
```

The app is accessible at `http://localhost:8082`.
