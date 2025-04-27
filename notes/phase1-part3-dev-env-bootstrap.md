# 🚀 Phase 1 – Part 3: Dev Environment Bootstrap (Final Engineering-Grade Version)

This document outlines the **production-grade dev environment** setup for the Application Security Pipeline project,  
focusing on **secure, reproducible, and maintainable** engineering best practices.

---

# 📦 Prerequisites

Install the following tools:

| Tool | Purpose |
|-----|---------|
| GitHub account | Manage repository and workflows |
| Git CLI | Version control |
| VSCode | Code editor |
| Go (Golang >=1.21) | Backend development |
| Podman OR Docker | Container building and running |
| Make | Local task automation |
| Node.js + npm | Frontend development |
| Python3 + pip | Pre-commit hooks management |

---

# 🐳 Pull and Run the Base Image for Local Testing

```bash
# Pull Go 1.24.2 on Alpine 3.21 base image
podman pull docker.io/library/golang:1.24.2-alpine3.21

# Run an interactive shell inside container
podman run -it --rm docker.io/library/golang:1.24.2-alpine3.21 /bin/sh
```

✅ We are testing our installation steps inside a clean environment matching production.

---

# 🛠️ Manual Version Control Testing (Simulate ARGs Locally)

Inside the container shell:

```bash
# Define pinned versions manually
SEMGREP_VERSION=1.120.0
TRIVY_VERSION=0.61.1
PYTHON3_VERSION=3.12.10-r0
PIP_VERSION=24.3.1-r0
GIT_VERSION=2.47.2-r0
CURL_VERSION=8.12.1-r1
JQ_VERSION=1.7.1-r0

# Update package manager index
apk update

# Install pinned versions of required packages
apk add --no-cache   python3=${PYTHON3_VERSION}   py3-pip=${PIP_VERSION}   git=${GIT_VERSION}   curl=${CURL_VERSION}   jq=${JQ_VERSION}

# Install Semgrep using pinned version
pip install semgrep==${SEMGREP_VERSION} --break-system-packages --root-user-action=ignore

# Download specific Trivy version archive
curl -L https://github.com/aquasecurity/trivy/releases/download/v0.61.1/trivy_0.61.1_Linux-64bit.tar.gz -o trivy.tar.gz
tar zxvf trivy.tar.gz
mv trivy /usr/local/bin/
```

✅ All installations follow strict version control, ensuring reproducibility.

---

# 📋 Verify Installed Tools

```bash
# Confirm versions match pinned requirements
go version
git --version
python3 --version
pip --version
semgrep --version
trivy --version
```

---

# 🧪 Real-World Testing of Scanners

Clone DVWA and run Semgrep and parse output with JQ:

```bash
git clone https://github.com/digininja/DVWA.git
semgrep --config=auto --json --output semgrep-dvwa.json ./DVWA
jq . semgrep-dvwa.json > semgrep-dvwa-new.json
```

Check Container ID and copy the semgrep-dvwa-new.json output to current local directory in new terminal:

```bash
podman ps
podman cp <container-id>:/go/semgrep-dvwa-new.json .
```

Pull vulnerable image and copy into container:

```bash
podman pull docker.io/library/python:3.7-alpine
podman save python:3.7-alpine -o python-3.7-alpine.tar
podman cp python-3.7-alpine.tar c7a3873:/go/
```

Scan vulnerable image in container and parse output in JQ:

```bash
trivy image --input python-3.7-alpine.tar --format json --output trivy-vuln.json
jq . trivy-vuln.json > trivy-vuln-new.json
```

Check Container ID and copy the trivy-vuln-new.json output to current local directory in new terminal:

```bash
podman ps
podman cp <container-id>:/go/trivy-vuln-new.json .
```

✅ Validates Semgrep and Trivy operation using real-world artifacts.

---

# 🛠️ Final Dockerfile (Engineering Best Practices)

```Dockerfile
# Start with your base image
FROM golang:1.24.2-alpine3.21

# Define ARGs immediately after FROM (not before)
ARG SEMGREP_VERSION=1.120.0
ARG TRIVY_VERSION=0.61.1
ARG PYTHON3_VERSION=3.12.10-r0
ARG PIP_VERSION=24.3.1-r0
ARG GIT_VERSION=2.47.2-r0
ARG CURL_VERSION=8.12.1-r1
ARG JQ_VERSION=1.7.1-r0

# Set pip environment to avoid warnings
ENV PIP_ROOT_USER_ACTION=ignore

# Update package manager and install pinned versions
RUN apk update && apk add --no-cache \
    python3=${PYTHON3_VERSION} \
    py3-pip=${PIP_VERSION} \
    git=${GIT_VERSION} \
    curl=${CURL_VERSION} \
    jq=${JQ_VERSION}

# Install Semgrep pinned version
RUN pip install semgrep==${SEMGREP_VERSION} --break-system-packages --root-user-action=ignore

# Install Trivy pinned version manually
RUN curl -L https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.tar.gz -o trivy.tar.gz \
    && tar zxvf trivy.tar.gz \
    && mv trivy /usr/local/bin/ \
    && rm -f trivy.tar.gz

# Set working directory
WORKDIR /app

# Default entrypoint
CMD ["/bin/sh"]
```

---

# 🛠️ Makefile (Automate Local Workflows)

```makefile
# Auto-detect GitHub details
GITHUB_REPO_URL=$(shell git config --get remote.origin.url)
GITHUB_ORG_NAME=$(shell echo $(GITHUB_REPO_URL) | sed -E 's/.*[:\/](.*)\/(.*)\.git/\1/')
APP_NAME=$(shell echo $(GITHUB_REPO_URL) | sed -E 's/.*[:\/](.*)\/(.*)\.git/\2/')
VERSION=0.1.0
IMAGE=ghcr.io/$(GITHUB_ORG_NAME)/$(APP_NAME)

# Build Docker image
build:
	docker build -t $(APP_NAME):$(VERSION) -f docker/Dockerfile .

# Start interactive shell inside built container
shell:
	docker run -it --rm -v $(PWD):/app -w /app $(APP_NAME):$(VERSION)

# Tag Docker image for registry upload
tag:
	docker tag $(APP_NAME):$(VERSION) $(IMAGE):$(VERSION)
	docker tag $(APP_NAME):$(VERSION) $(IMAGE):latest

# Push Docker image to GHCR
push:
	docker push $(IMAGE):$(VERSION)
	docker push $(IMAGE):latest

# Run Go unit tests inside container
test-container:
	docker run --rm -v $(PWD):/app -w /app $(APP_NAME):$(VERSION) go test ./...

# Run Semgrep scanner standalone
test-sast:
	semgrep --config=auto --json --output semgrep-test.json ./testdata

# Run Trivy scanner standalone
test-trivy:
	trivy image --format json --output trivy-test.json alpine:latest

# Full app scan
test-app:
	go build -o scanner ./cmd/scanner
	./scanner --mode sast --path ./testdata
	./scanner --mode container --image alpine:latest
```

---

# 🚀 GitHub Actions Workflow to Publish Docker Base Image

`.github/workflows/docker-publish.yml`

```yaml
name: base-image

on:
  push:
    tags:
      - "v*.*.*"  # Only trigger on tagged releases

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
    - uses: actions/checkout@v4

    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and Push Docker Image
      run: |
        IMAGE_NAME=ghcr.io/${{ github.repository }}
        VERSION=${GITHUB_REF#refs/tags/v}
        docker build -t $IMAGE_NAME:$VERSION -t $IMAGE_NAME:latest -f docker/Dockerfile .
        docker push $IMAGE_NAME:$VERSION
        docker push $IMAGE_NAME:latest
```

---

# 🔄 RenovateBot Config to Watch Base Image Updates

`.github/renovate.json`

```json
{
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchDatasources": ["docker"],
      "matchPackageNames": ["golang"],
      "matchFileNames": ["docker/Dockerfile"],
      "groupName": "Base image updates"
    }
  ]
}
```

---

# 🚀 Final Step: Merge and Release First DevOps Version

Merge this feature branch into `main`.

Tag the repository for the first release:

```bash
git checkout main
git pull origin main
git tag v0.0.1
git push origin v0.0.1
```

This will trigger GitHub Actions to build and push your Docker image to GitHub Container Registry (GHCR).