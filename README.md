# MEAN Stack CRUD Application — DevOps Assignment

A full-stack CRUD application built with **MongoDB, Express, Angular, and Node.js**, fully containerized with Docker and deployed using a CI/CD pipeline.

---

## Project Structure

```
.
├── backend/                  # Node.js + Express REST API
│   ├── app/
│   │   ├── config/db.config.js
│   │   ├── models/
│   │   └── routes/
│   ├── server.js
│   ├── package.json
│   ├── Dockerfile
│   └── .dockerignore
├── frontend/                 # Angular 15 SPA
│   ├── src/
│   ├── nginx.conf            # Nginx config for Angular routing
│   ├── Dockerfile
│   └── .dockerignore
├── nginx/
│   └── nginx.conf            # Main reverse proxy config
├── .github/
│   └── workflows/
│       └── ci-cd.yml         # GitHub Actions CI/CD pipeline
├── docker-compose.yml        # Local/dev (builds from source)
├── docker-compose.prod.yml   # Production (pulls from Docker Hub)
└── README.md
```

---

## Architecture

```
User (Browser)
      |
      v
  Nginx (Port 80)  <--- Single entry point (Reverse Proxy)
      |
      |-- /api/*  -----------> Backend (Node.js :5000)
      |                              |
      `-- /*  --------------> Frontend (Angular/Nginx :80)
                                     |
                               MongoDB (:27017)
```

All services run in Docker containers on a shared bridge network. MongoDB data is persisted via a named Docker volume.

---

## Prerequisites

- Docker and Docker Compose installed
- Git
- A Docker Hub account
- An Ubuntu VM (AWS EC2, Azure, etc.)

---

## Local Development Setup

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
```

### 2. Build and Run
docker compose up --build
```

### 3. Access

Open: **http://localhost**

---

## VM Deployment (Ubuntu)

### Step 1: Provision an Ubuntu 22.04 VM

Open **port 80** in your cloud security group/firewall.

### Step 2: Install Docker on the VM
sudo apt update && sudo apt upgrade -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
docker --version
docker compose version
```

### Step 3: Clone Repo on VM

git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
```

### Step 4: Deploy with Production Compose

export DOCKER_USERNAME=your-dockerhub-username
docker compose -f docker-compose.prod.yml up -d
```

### Step 5: Verify
docker ps
# nginx, frontend, backend, mongo should all be running
```

Open `http://<VM-PUBLIC-IP>` in your browser.

---

## Docker Images

### Build Locally
docker build -t your-dockerhub-username/mean-backend:latest ./backend
docker build -t your-dockerhub-username/mean-frontend:latest ./frontend
```

### Push to Docker Hub
docker login
docker push your-dockerhub-username/mean-backend:latest
docker push your-dockerhub-username/mean-frontend:latest
```

---

## CI/CD Pipeline — GitHub Actions

Triggers on every push to `main`. Defined in `.github/workflows/ci-cd.yml`.

### Required GitHub Secrets

Go to: **GitHub Repo → Settings → Secrets and variables → Actions**

| Secret Name       | Description                              |
|-------------------|------------------------------------------|
| `DOCKER_USERNAME` | Docker Hub username                      |
| `DOCKER_PASSWORD` | Docker Hub password or access token      |
| `VM_HOST`         | Public IP of your Ubuntu VM             |
| `VM_USER`         | SSH username (e.g., `ubuntu`)           |
| `VM_SSH_KEY`      | Private SSH key content (from .pem file)|

### Pipeline Stages

```
Push to main
     |
     v
[Job 1: Build & Push]
  - Checkout code
  - Login to Docker Hub
  - Build & push backend:latest
  - Build & push frontend:latest
     |
     v
[Job 2: Deploy to VM]
  - Copy compose files to VM via SCP
  - SSH into VM
  - Pull latest images from Docker Hub
  - docker compose down
  - docker compose -f docker-compose.prod.yml up -d
  - Prune old images
```

---

## Nginx Reverse Proxy

**Main Nginx** (`nginx/nginx.conf`) — single entry point on port 80:
- `/api/*` → proxied to backend container (port 5000)
- `/*` → proxied to frontend container (port 80)

**Frontend Nginx** (`frontend/nginx.conf`) — handles Angular client-side routing:
- All paths redirect to `index.html` so Angular Router works correctly

---

## Environment Variables

| Variable    | Service  | Default                          | Description           |
|-------------|----------|----------------------------------|-----------------------|
| `PORT`      | Backend  | `5000`                           | Node.js server port   |
| `MONGO_URI` | Backend  | `mongodb://mongodb:27017/dd_db`  | MongoDB connection URL |

---

## Stopping the Application

# Stop containers (keeps data)
docker compose down

# Stop and remove all data
docker compose down -v

