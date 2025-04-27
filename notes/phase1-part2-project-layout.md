# 🧱 Phase 1 – Part 2: Project Layout and Initial Structure

This document defines the real-world, secure, and scalable structure for the Application Security Pipeline project.

---

# 📋 Agreed Updated High-Level Directory Structure

```plaintext
application-security-pipeline/
├── cmd/scanner/         # CLI entrypoint, flags, config parsing
├── internal/
│   ├── cli/             # CLI argument parsing and validation
│   ├── dispatcher/      # Central orchestration: CLI -> Scanner -> Enforcement -> DB
│   ├── scanner/
│   │   ├── sast/        # Semgrep integration (real JSON scanning)
│   │   ├── container/   # Trivy integration (tarball scan in dev, image scan in prod)
│   ├── storage/         # GORM + SQLite database storage
│   ├── models/          # Core domain entities (Finding, ScanResult, etc.)
│   ├── config/          # YAML config loader, criticality classification
│   └── api/             # RESTful API for frontend dashboard
├── pkg/types/           # Shared enums: Severity, ScanMode, EnforcementMode, Criticality
├── docker/              # Dockerfile with Semgrep + Trivy pre-installed
├── web/                 # Frontend (React + Tailwind + Vite)
├── .github/workflows/   # GitHub Actions CI/CD workflows
├── .pre-commit-config.yaml
├── Makefile
├── go.mod / go.sum
```

---

# 🛠️ Bootstrap Commands to Create Initial Structure

```bash
# Create base folder and initialize Go module
mkdir -p application-security-pipeline && cd application-security-pipeline
go mod init github.com/jdmansec/application-security-pipeline

# Create all top-level and essential directories
mkdir -p {cmd/scanner,internal/{cli,dispatcher,scanner/{sast,container},storage,models,config,api},pkg/types,docker,web}
```

> ⚠️ Files are created **only as needed** during each phase (strict compile-safe order).

---

# 📦 Module Responsibilities Overview

## `cmd/scanner`
- CLI main entry
- Handles flags like `--mode`, `--path`, `--image`, `--version`
- Passes validated input to dispatcher

## `internal/cli`
- Parses and validates CLI arguments
- Adds support for `--version` flag via `ldflags`

## `internal/dispatcher`
- Receives CLI input
- Decides whether to run Semgrep or Trivy based on mode
- Applies enforcement mode (secure/balanced/warning)
- Sends findings to DB

## `internal/scanner/sast`
- Runs Semgrep scans (real source scan)
- Parses Semgrep JSON output
- Converts to internal `Finding` model

## `internal/scanner/container`
- Runs Trivy scans
- In Dev: scans tarball filesystem exports
- In Prod: scans built Docker images directly
- Parses Trivy JSON output

## `internal/storage`
- Manages GORM + SQLite database
- Handles migrations (auto-create tables)

## `internal/models`
- Defines Go structs matching DB tables and JSON schemas
- (e.g., Finding, Repository, SuppressionLog, ScanResult)

## `internal/config`
- Loads `config.yaml`
- Applies defaults
- Reads GitHub Topics to classify project criticality
- Maps Criticality -> EnforcementMode

## `internal/api`
- REST API server (Echo / Chi framework)
- Exposes project, findings, suppression, and SLA endpoints

## `pkg/types`
- Defines enums and shared constants:
  - Severity (LOW, MEDIUM, HIGH, CRITICAL, BANNED)
  - ScanMode (SAST, CONTAINER)
  - EnforcementMode (warning, balanced, secure)
  - Criticality (mission-critical, business-critical, operational, administrative)

---

# 🐳 Updated Dockerfile Strategy (Example)

```Dockerfile
# docker/Dockerfile
FROM golang:1.24.0-alpine3.19

RUN apk add --no-cache git python3 py3-pip curl jq
RUN pip install semgrep
RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

WORKDIR /app
CMD ["/bin/sh"]
```

✅ Go, Semgrep, Trivy pre-installed.  
✅ Minimal, secure base image.  
✅ No root user required.

> ⚡ In dev, we scan `.tar` images with Trivy. In production, real Docker images are scanned directly.

---

# 🧪 Updated Test Layout

Tests colocated next to logic:

```plaintext
internal/
├── cli/cli_test.go
├── dispatcher/dispatcher_test.go
├── scanner/sast/sast_test.go
├── scanner/container/container_test.go
├── storage/storage_test.go
├── config/config_test.go
```

✅ Full unit test coverage of CLI, Dispatcher, Semgrep Scanner, Trivy Scanner, Storage, and Config Loader.

---

# ✅ Summary

✅ Secure-by-default project bootstrapped  
✅ Compile-safe file and directory creation only during feature phases  
✅ Real Semgrep + Trivy scanning integrated  
✅ Clear Dockerfile strategy for Dev vs Prod  
✅ Proper criticality-based enforcement logic planned  
✅ Full GitHub Actions CI/CD preparation

---

Next Step: Begin `phase1-part3-dev-env-bootstrap.md`