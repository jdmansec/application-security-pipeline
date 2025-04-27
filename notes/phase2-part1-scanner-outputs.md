# 🚀 Phase 1 – Part 4: Scanner Outputs

In this phase, we:
- Validate that Semgrep and Trivy run correctly inside your dev container
- Capture real-world JSON outputs
- Analyze the outputs to define correct Go models for future phases

✅ No stubs, no fakes  
✅ Real environment, real tools, real results

---

# 📋 Why We Capture Real Outputs First

| Reason | Why |
|--------|-----|
| ✅ Build correct Go structs later | So fields match real-world data |
| ✅ Confirm tools are installed and work inside container | No "works on my laptop" issues |
| ✅ Save realistic findings to build real parsers | No broken assumptions |

---

# 🛠 Step 1: Build and Enter Your Project Container

From your project root:

```bash
make build   # Build the dev container // docker build -t $(APP_NAME):$(VERSION) -f docker/Dockerfile .
make shell   # Enter interactive shell inside the container // docker run -it --rm -v $(PWD):/app -w /app $(APP_NAME):$(VERSION)
```

✅ You are now inside your project environment.

---

# 🧪 Step 2: Clone a Vulnerable App (DVWA)

Inside the container shell:

```bash
git clone https://github.com/digininja/DVWA.git
```

✅ DVWA source code is now at `/app/DVWA`.

---

# 🛠 Step 3: Run Semgrep on DVWA (Save Output)

Inside the container shell:

```bash
semgrep --config=auto --json --output semgrep-dvwa.json ./DVWA
```

✅ Semgrep JSON output is saved at `/app/semgrep-dvwa.json`.

---

# 🐳 Step 4: Pull and Scan an Old Vulnerable Docker Image

## 4.1 Pull and Save the Image as Tarball (On Your Host)

Outside the container:

```bash
podman pull python:3.7-alpine
podman save python:3.7-alpine -o python-3.7-alpine.tar
```

✅ This saves the entire Docker image into a `.tar` file.

## 4.2 Copy the Tarball into the Container

Find your running container ID (example):

```bash
podman ps
podman cp python-3.7-alpine.tar container_id:/app/
```

✅ Now `/app/python-3.7-alpine.tar` exists inside your container.

## 4.3 Scan the Tarball with Trivy (Inside Container)

Inside the container shell:

```bash
trivy image --input python-3.7-alpine.tar --format json --output trivy-vuln.json
```

✅ Trivy JSON output is saved at `/app/trivy-vuln.json`.

---

# 🔥 Important: Note for Production

> ⚠️ In real scanner deployments (not inside dev container), you will scan **live Docker images**, not tarballs.

Example real scanner usage:

```bash
trivy image my-webapp:latest
```

✅ Trivy will scan built Docker images directly, not filesystem tarballs.

✅ Tarball scanning is **only for dev testing inside this dev container**, because Docker socket is not available.

---

# 🔍 Step 5: Analyze Semgrep Output (Example Fields)

Example from `semgrep-dvwa.json`:

```json
{
  "results": [
    {
      "check_id": "python.lang.correctness.useless-comparison",
      "path": "DVWA/login.php",
      "start": { "line": 42 },
      "extra": {
        "message": "Possible useless comparison",
        "severity": "WARNING"
      }
    }
  ]
}
```

✅ Fields to extract:
- `check_id`
- `path`
- `start.line`
- `extra.message`
- `extra.severity`

---

# 🔍 Step 6: Analyze Trivy Output (Example Fields)

Example from `trivy-vuln.json`:

```json
{
  "Results": [
    {
      "Target": "python:3.7-alpine",
      "Vulnerabilities": [
        {
          "VulnerabilityID": "CVE-2019-1549",
          "PkgName": "openssl",
          "InstalledVersion": "1.1.1",
          "Severity": "HIGH",
          "Description": "OpenSSL vulnerability..."
        }
      ]
    }
  ]
}
```

✅ Fields to extract:
- `VulnerabilityID`
- `PkgName`
- `InstalledVersion`
- `Severity`
- `Description`

---

# 📦 Save These Files for Future Parsing

Inside `/app/`, you should have:

```bash
semgrep-dvwa.json
trivy-vuln.json
python-3.7-alpine.tar
```

✅ These will be used to design your Go structs and scanner parsers.

---

# ✅ Final Summary

| Task | Status |
|------|--------|
| Build and shell into container | ✅ |
| Clone DVWA | ✅ |
| Scan DVWA with Semgrep | ✅ |
| Save old vulnerable image as tar | ✅ |
| Copy tarball into container | ✅ |
| Scan tarball with Trivy | ✅ |
| Capture and study real JSON outputs | ✅ |

You are now perfectly ready to move on to **Phase 5: Go Models and Real Scanner Parsers**! 🚀

---