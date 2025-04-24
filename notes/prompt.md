# 🧾 Application Security Pipeline – Complete Project Prompt

> I want to build a **secure production-grade Application Security Scanner** using **Go (Golang)** for the backend and **TypeScript + React** for the frontend. The scanner will scan source code and container images using different modes, enforce security policies, track posture over time, and be usable via the CLI, GitHub Actions, and a web UI.
>
> The implementation must:
> - Follow **logical, compile-safe order** (define everything before use)
> - Use **semantic versioning** (`v0.0.1`+)
> - Follow **secure-by-default engineering practices**
> - Meet a **NASA-grade quality bar**: modular, testable, maintainable, and documented

---

## ✅ Core Features

### 📦 Scan Modes (Phase 1+)
- **SAST Mode**: Implement with Semgrep
- **Container Mode**: Implement CLI + arch now, Trivy integration later (stub scanner returns dummy results)

### 🧱 Project Structure (with CLI → Dispatcher → Scanner architecture)
```plaintext
security-scanner/
├── cmd/scanner/         # CLI: entrypoint, flags, config parsing
├── internal/
│   ├── cli/             # CLI logic layer
│   ├── dispatcher/      # Orchestration: receives CLI input, routes to scanner, applies policy
│   ├── scanner/         # Scan runners by mode (SAST/Container)
│   │   ├── sast/        # Semgrep logic
│   │   ├── container/   # Trivy placeholder logic
│   ├── storage/         # GORM + SQLite
│   ├── models/          # Repository, Finding, ScanResult, etc.
│   ├── config/          # Load config.yaml, enforce defaults
│   └── api/             # RESTful API for frontend
├── pkg/types/           # Shared enums: Severity, ScanMode, EnforcementMode
├── docker/              # Custom Dockerfile
├── web/                 # Frontend: TypeScript + React + Tailwind + Vite
├── .github/workflows/   # GitHub Actions
├── .pre-commit-config.yaml
├── Makefile
├── go.mod / go.sum
```

### ⚙️ CLI Flow
- `scanner --mode sast --path ./src`
- `scanner --mode container --image my-app:latest`
- **Flow:** CLI → Dispatcher → Scanner → DB → Policy engine

---

## 🔐 Enforcement Modes
Three levels defined in `EnforcementMode`:
1. `secure`: All findings block pipeline
2. `balanced`: Only HIGH, CRITICAL block pipeline
3. `warning` (default): Only BANNED blocks, all others advisory

---

## ⚠️ Configurable Alert Thresholds
Defined in `config.yaml` or flags (not blocking):
```yaml
thresholds:
  critical: 0
  high: 0
  medium: 5
  low: 10
```
- Alert-only. Pipeline block behaviour comes from enforcement mode.

---

## ⛔ Suppression Handling
- Findings can be `Suppressed` with a reason
- Suppressions expire after 90 days automatically
- Immutable suppression log (`SuppressionLog`):
  - `FindingID`, `SuppressedBy`, `Reason`, `SuppressedAt`, `ExpiresAt`
  - No updates or deletes allowed

---

## 🔁 SLA Timelines (Tracked for Reporting)
- CRITICAL: 7 days
- HIGH: 30 days
- MEDIUM: 60 days
- LOW: 90 days

Tracked by `FirstSeenAt` and `ResolvedAt`.

---

## 📊 Frontend Features (React + Tailwind + Vite)
- Dashboard per project
- Findings table with filtering by severity, status, suppression
- SLA breach indicators
- Suppression stats
- Trends over time (new vs resolved findings)
- Powered by Go API

---

## 🛠️ DevOps + CI/CD Integration

### Docker Image
- Custom image with Go, SQLite, Semgrep
- Secure, minimal, no root
- Trivy added in future phase

### Pre-commit Hooks (`.pre-commit-config.yaml`)
- `go fmt`, `goimports`, `go vet`, `go test`, `semgrep`

### Makefile (cross-platform)
- `make build`, `make test`, `make scan`, `make docker`, `make lint`

### GitHub Actions
- Run scanner in PRs and on `main`:
  - `make test`
  - `make scan`
  - Fail if blocked
  - Summary in job output

---

## 🧪 Testing Strategy
- Full unit tests for all models, scanner logic, enforcement logic
- Run with: `go test ./...`
- Container mode uses mocked/dummy findings

---

## 🔍 Real-World Testing (DVWA)
- Clone DVWA
- Run scanner via CLI and GitHub Actions
- Check results in logs and frontend
- Verify suppressions, SLA breaches, and trend data

---

## 📦 Additional Requirements (MANDATORY)

| Feature                             | Required | Notes |
|------------------------------------|----------|-------|
| 🔐 Immutable suppression logs       | ✅       | Use SuppressionLog, never allow edits/deletes |
| ⚙️ Configurable alert thresholds     | ✅       | For all severities via config.yaml |
| 📈 Frontend trend charts             | ✅       | Use Recharts or Chart.js |
| 📦 CLI `--version` flag              | ✅       | Embed via `-ldflags`, print with `--version` |
| 🧪 CLI tests in GitHub Actions      | ✅       | Run `make test` and `make scan` in CI job |

---

## 🧠 Development Philosophy
This project should be written as if it's powering a NASA security system:
- Always compile-safe, test-first, and modular
- All logic idiomatic Go
- Frontend and backend must remain decoupled but in sync
- Data must be structured, auditable, and visualised across time

Implement everything in strict sequence with every module self-contained and testable before proceeding.
---

## 🏷️ Project Criticality Classification (New Feature)

To ensure enforcement policy scales with business importance, each project will be classified based on **system criticality**, inferred via GitHub repository topics.

### 🎯 Levels of Criticality

1. `Mission-critical` – Essential to core business or life/safety systems
2. `Business-critical` – Key revenue-generating or external-facing applications
3. `Business-operational` – Internal apps needed for daily operations
4. `Administrative` – Low-risk or support utilities

### 🔄 GitHub Tag-Based Detection

Projects must define a GitHub topic to indicate criticality:
- `criticality:mission`
- `criticality:business-critical`
- `criticality:business-operational`
- `criticality:administrative`

### 🔐 Forced EnforcementMode Mapping

| Criticality Level     | Enforced `EnforcementMode` |
|-----------------------|----------------------------|
| Mission-critical       | `secure`                   |
| Business-critical      | `balanced`                 |
| Business-operational   | `warning`                  |
| Administrative         | `warning`                  |

This approach ensures automated security posture enforcement based on project sensitivity.