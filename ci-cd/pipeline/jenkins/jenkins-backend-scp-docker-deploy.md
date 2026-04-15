# Jenkins Backend Deployment Guide (SCP + Docker)

This document explains **backend deployment only** for a Spring Boot application using Jenkins Freestyle jobs, SCP, and Docker.

It focuses on:
- transferring backend artifacts with SCP
- using Dockerfile to build runtime images
- executing deploy.sh on the remote server

---

## 1. Backend Deployment Mental Model

```text
Jenkins
  ├── mvn clean package
  └── target/*.jar
        ↓
scp
        ↓
Remote server (/opt/web-demo-backend)
        ↓
Docker build (JDK runtime)
        ↓
Docker run
```

Key rule:

> **Jenkins builds, the server only runs Docker.**

---

## 2. Backend Build Output

After Jenkins runs:

```bash
mvn clean package
```

The build output is:

```text
target/
└── web-demo-backend-<version>.jar
```

This JAR is the **only backend artifact** that needs to be deployed.

---

## 3. Managed Config Files (Backend)

Backend deployment usually uses **Config File Provider** with these managed files:

| Purpose | Jenkins Variable | Final Filename on Server |
|------|-----------------|--------------------------|
| Deploy script | BACKEND_DEPLOY_SH | deploy.sh |
| Dockerfile | BACKEND_DOCKER_FILE | Dockerfile |

Important:

> Jenkins exposes these files as **temporary paths**, referenced only by variables.

---

## 4. SCP Backend Artifacts and Config Files

Example Jenkins **Execute shell** step:

```bash
set -e

APP_DIR=/opt/web-demo-backend
SERVER_USER=deploy
SERVER_IP=SERVER_IP

# ensure target directory exists
ssh ${SERVER_USER}@${SERVER_IP} "mkdir -p ${APP_DIR}"

# copy backend JAR (rename to app.jar)
scp target/web-demo-backend-*.jar   ${SERVER_USER}@${SERVER_IP}:${APP_DIR}/app.jar

# copy managed deploy script
scp "$BACKEND_DEPLOY_SH"   ${SERVER_USER}@${SERVER_IP}:${APP_DIR}/deploy.sh

# copy managed Dockerfile
scp "$BACKEND_DOCKER_FILE"   ${SERVER_USER}@${SERVER_IP}:${APP_DIR}/Dockerfile
```

Notes:
- SCP renames the JAR to a fixed name (`app.jar`)
- Jenkins temp paths are never exposed on the server

---

## 5. Execute Backend Deployment on Server

After files are transferred, Jenkins triggers deployment:

```bash
ssh ${SERVER_USER}@${SERVER_IP} << EOF
  set -e
  chmod +x ${APP_DIR}/deploy.sh
  cd ${APP_DIR}
  ./deploy.sh
EOF
```

This runs **entirely on the server**.

---

## 6. Typical backend deploy.sh Responsibilities

A typical `deploy.sh` performs:

- stop old container
- remove old container
- build Docker image
- run new container

Example:

```bash
#!/usr/bin/env bash
set -e

APP_NAME=web-demo-backend
IMAGE_NAME=web-demo-backend:latest
NETWORK_NAME=web-demo-net

docker stop $APP_NAME || true
docker rm $APP_NAME || true

docker build -t $IMAGE_NAME .

docker run -d \
  --name $APP_NAME \
  --network $NETWORK_NAME \
  -p 8080:8080 \
  --restart unless-stopped \
  $IMAGE_NAME
```

---

## 7. Why chmod +x Is Required

Files copied via SCP usually have permissions:

```text
-rw-r--r--
```

Shell scripts are **not executable** by default, so Jenkins must run:

```bash
chmod +x deploy.sh
```

---

## 8. Error Handling Rule

All scripts use:

```bash
set -e
```

Meaning:
- any command failure stops execution
- prevents half-deployed backend services

---

## 9. Common Mistakes to Avoid

- Running Maven on the server ❌
- SCPing multiple JAR versions ❌
- Editing Dockerfile directly on the server ❌
- Forgetting to stop old containers ❌

---

## 10. Final Takeaway

> Backend deployment with Jenkins should follow a strict separation: Jenkins builds the JAR, SCP delivers it, and Docker runs it on the server.

---

**End of backend deployment guide**
