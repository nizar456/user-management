# CI/CD Pipeline Guide - User Management Project

This guide explains how to run the CI/CD pipeline with GitHub Actions, SonarQube, and Docker.

---

## 📋 Prerequisites

Before running the pipeline, ensure the following services are running on your local machine:

| Service                | URL                   | Purpose                          |
| ---------------------- | --------------------- | -------------------------------- |
| **SonarQube**          | http://localhost:9000 | Code quality analysis            |
| **Docker Desktop**     | -                     | Build container images           |
| **Self-Hosted Runner** | -                     | Execute GitHub Actions workflows |

---

## 🚀 Step 1: Start SonarQube

SonarQube must be running before pushing code.

Start your SonarQube service and verify it's running:

```powershell
# Check if SonarQube is accessible
Invoke-WebRequest -Uri "http://localhost:9000/api/system/status" -UseBasicParsing
```

Expected response: `{"status":"UP"}`

---

## 🏃 Step 2: Start the Self-Hosted Runner

The GitHub Actions runner must be running locally to execute workflows.

### Start the Runner

```powershell
# Navigate to the runner folder
cd "C:\studySpace\module obj & test logiciel\user-management\actions-runner"

# Start the runner (keep this terminal open!)
.\run.cmd
```

### Expected Output

```
√ Connected to GitHub
Current runner version: '2.330.0'
2025-12-25 16:38:09Z: Listening for Jobs
```

### Start Runner in Background (New Window)

```powershell
Start-Process powershell -ArgumentList "-NoExit", "-Command", "cd 'C:\studySpace\module obj & test logiciel\user-management\actions-runner'; .\run.cmd"
```

---

## 📤 Step 3: Push Code to Trigger the Pipeline

### Normal Push (with changes)

```powershell
cd "C:\studySpace\module obj & test logiciel\user-management"

git add .
git commit -m "Your commit message"
git push
```

### Fake Push (trigger pipeline without code changes)

```powershell
cd "C:\studySpace\module obj & test logiciel\user-management"

git commit --allow-empty -m "Trigger CI/CD pipeline"
git push
```

---

## 🔄 Complete Workflow (All Steps)

Run these commands in order:

```powershell
# 1. Start SonarQube (if using docker-compose)
cd "C:\studySpace\module obj & test logiciel\user-management\local-environment"
docker-compose up -d

# 2. Start the self-hosted runner in a new window
Start-Process powershell -ArgumentList "-NoExit", "-Command", "cd 'C:\studySpace\module obj & test logiciel\user-management\actions-runner'; .\run.cmd"

# 3. Wait a few seconds for runner to connect, then trigger the pipeline
Start-Sleep -Seconds 5
cd "C:\studySpace\module obj & test logiciel\user-management"
git commit --allow-empty -m "Trigger CI/CD pipeline"
git push
```

---

## 📊 View Results

| What                   | Where                                               |
| ---------------------- | --------------------------------------------------- |
| **Workflow Runs**      | https://github.com/nizar456/user-management/actions |
| **SonarQube Analysis** | http://localhost:9000/dashboard?id=user-management  |

---

## 🔧 Troubleshooting

### Runner Not Picking Up Jobs

```powershell
# Stop any existing runner processes
Stop-Process -Name "Runner.Listener" -Force -ErrorAction SilentlyContinue
Stop-Process -Name "Runner.Worker" -Force -ErrorAction SilentlyContinue

# Restart the runner
cd "C:\studySpace\module obj & test logiciel\user-management\actions-runner"
.\run.cmd
```

### SonarQube Connection Refused

```powershell
# Check if SonarQube is running
Invoke-WebRequest -Uri "http://localhost:9000/api/system/status" -UseBasicParsing -TimeoutSec 5
```

If not running, start it with Docker Compose.

### Token Invalid/Expired

1. Go to http://localhost:9000/account/security
2. Generate a new token
3. Update the token in:
   - `.github/workflows/build-an-deploy.yml` (in the Maven command)
   - `sonar-project.properties`

---

## 📁 Project Structure

```
user-management/
├── .github/
│   └── workflows/
│       └── build-an-deploy.yml    # CI/CD workflow definition
├── actions-runner/                 # Self-hosted GitHub runner
│   └── run.cmd                     # Start the runner
├── local-environment/
│   └── docker-compose.yml          # SonarQube + dependencies
├── sonar-project.properties        # SonarQube configuration
├── pom.xml                         # Maven build configuration
├── Dockerfile                      # Docker image definition
└── src/                            # Application source code
```

---

## 🔄 Pipeline Steps

When you push code, the pipeline executes:

```
1. Checkout Code
      ↓
2. Set up JDK 21
      ↓
3. Maven Build + Tests + SonarQube Analysis
      ↓
4. Docker Build (carikk/quality-gate:latest)
```

---

## ⚠️ Important Notes

1. **Keep the runner terminal open** - If you close it, workflows will queue and wait
2. **SonarQube must be running** before pushing code
3. **Docker Desktop must be running** for the Docker build step
4. The runner only processes jobs for the `main` branch (as configured in the workflow)
