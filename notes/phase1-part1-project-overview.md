# 🚀 Phase 1 – Part 1: Project Overview

## 🛠️ Project Prerequisites

Before starting, ensure your system has the following tools installed:

| Tool | Minimum Version | Purpose |
|-----|------------------|---------|
| GitHub account | N/A | Manage repository, GitHub Actions |
| Git CLI | Latest stable | Clone repositories, manage code |
| VSCode | Latest stable | IDE for Go + TypeScript development |
| Go (Golang) | >= 1.21 | Backend development (CLI/API) |
| Podman OR Docker | Latest stable | Build and run project containers |
| Make | Latest stable | Automate common tasks |
| Node.js + npm | Latest LTS | Build and run React + TypeScript frontend |
| Python3 + pip | Latest stable | Pre-commit hooks locally |

---

## 📋 How to Check and Install Prerequisites

### ✅ Check Commands

```bash
git --version
go version
podman --version
docker --version
make --version
node -v && npm -v
python3 --version && pip3 --version
```

---

### 🛠️ Install Instructions (Platform-Specific)

| Tool | Install on WSL/Ubuntu | Install on Windows (Git Bash/PowerShell with Chocolatey) |
|------|-----------------------|---------------------------------------------------------|
| Git | `sudo apt install git` | `choco install git` |
| Go | `sudo apt install golang` or from [golang.org](https://golang.org/dl/) | `choco install golang` |
| Podman | `sudo apt install podman` | `choco install podman` |
| Docker | Follow steps below | `choco install docker-cli` |
| Make | `sudo apt install make` | `choco install make` |
| Node.js + npm | `sudo apt install nodejs npm` or use nvm | `choco install nodejs` |
| Python3 + pip | `sudo apt install python3 python3-pip` | `choco install python` |

> ⚡ Chocolatey Installation Guide: [https://chocolatey.org/install](https://chocolatey.org/install)

---

## 🐳 Installing Docker on Ubuntu WSL2 (Proper Way)

```bash
# Update system
sudo apt update && sudo apt upgrade

# Install required packages
sudo apt install --no-install-recommends apt-transport-https ca-certificates curl gnupg2

# Add Docker GPG key
. /etc/os-release
curl -fsSL https://download.docker.com/linux/${ID}/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc

# Add Docker stable repository
echo "deb [arch=amd64] https://download.docker.com/linux/${ID} ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list

# Update apt again
sudo apt update

# Install Docker CE
sudo apt install docker-ce docker-ce-cli containerd.io
```

✅ Official Docker install for WSL Ubuntu — not relying on Docker Desktop.

---

# 🧾 Objective

Build a **secure, production-grade Application Security Pipeline** that:

- Scans source code (Semgrep)
- Scans container images (Trivy)
- Enforces customizable policies
- Tracks posture over time
- Provides CLI-first access, GitHub Actions automation, and frontend dashboards
- Uses secure Go backend and TypeScript React frontend

---

# 📐 Architecture Overview

```plaintext
CLI → Dispatcher → Scanner (SAST/Container) → Storage → Policy Engine → API → Frontend
```

---

# 🧱 Project Structure

(standard corrected directory structure — same as previous)

---

# ✅ Phase 1 Deliverables (MVP Scope)

- CLI-driven scans
- Semgrep and Trivy real output parsing
- Enforcement modes with blocking behavior
- Full database tracking of findings, suppressions, SLA breaches
- CI/CD integration (GitHub Actions + Makefile)
- Frontend scaffolding (React, Tailwind, Vite)

---

# 🔐 Enforcement Modes

Includes `BANNED` overlay.

| Mode      | Blocks on |
|-----------|-----------|
| secure    | CRITICAL, HIGH, MEDIUM, LOW, BANNED |
| balanced  | CRITICAL, HIGH, BANNED |
| warning   | Only BANNED |

---

# ⚠️ Configurable Alert Thresholds

```yaml
thresholds:
  critical: 0
  high: 0
  medium: 5
  low: 10
```

---

# ⛔ Suppression Handling

- Suppress findings with a valid reason
- Automatic expiry after 90 days
- Immutable audit log of suppressions

---

# 🔁 SLA Timelines

| Severity  | SLA Days |
|-----------|----------|
| CRITICAL  | 7 |
| HIGH      | 30 |
| MEDIUM    | 60 |
| LOW       | 90 |

---

# 📊 Frontend Features

- Project security dashboards
- Filterable findings table
- Suppression tracking
- SLA breach monitoring
- Trend graphs over time

---

# 🛠️ DevOps and CI/CD

- Secure minimal Dockerfile
- GitHub Actions pipelines
- Pre-commit hooks (Python3 + pip)
- Full Makefile support

---

# 🧪 Testing Strategy

- Unit tests for all major components
- Real-world testing with DVWA (Semgrep) and old images (Trivy)
- Full compile-safe development flow

---

# 📦 Critical Requirements

| Feature | Required |
|---------|----------|
| Immutable suppression log | ✅ |
| Configurable alert thresholds | ✅ |
| Trend chart visualization | ✅ |
| CLI version metadata (`--version`) | ✅ |
| GitHub Actions CI enforcement | ✅ |

---

# 🔧 Special Trivy Behavior Clarification

| Environment | Trivy Scans |
|-------------|-------------|
| Dev container (testing) | Tarball images |
| Production scanner | Live Docker images |

---

# 🚀 Next Step

Proceed to Phase 1 Part 2: `phase1-part2-project-layout.md`