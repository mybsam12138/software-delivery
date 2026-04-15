# Jenkins Frontend Deployment: SCP + Dockerfile + deploy.sh

## 1. Purpose of This Section

This section explains **what to do after the frontend `dist/` directory is built**:
- how Jenkins uses **SCP** to transfer artifacts
- how **Dockerfile** and **nginx.conf** are delivered
- how **deploy.sh** is executed on the remote server

This applies to **Freestyle Jenkins jobs** using **Config File Provider**.

---

## 2. Key Mental Model

```text
Jenkins workspace
  └── dist/                ← frontend build output

Config File Provider
  ├── Dockerfile
  ├── nginx.conf
  └── deploy.sh
        ↓
scp (copy files)
        ↓
Remote server (/opt/web-demo-frontend)
        ↓
deploy.sh
  ├── docker build
  ├── docker stop/rm
  └── docker run
```

---

## 3. Using SCP in Jenkins (Why & How)

### Why SCP is used
- Simple
- Secure (SSH-based)
- No extra agent or service needed
- Industry-standard for small/medium deployments

### What Jenkins SCP actually does
- Jenkins runs `scp` from **inside the Jenkins container**
- Files are copied over SSH to the remote server
- SSH authentication uses Jenkins credentials

---

## 4. Config File Provider and Environment Variables

In Jenkins, you configured **Managed Files**:

| Managed File | Variable |
|-------------|----------|
| deploy-web-demo-frontend | `DEPLOY_SH` |
| docker-web-demo-frontend | `DOCKER_FILE` |
| nginx-web-demo-frontend | `NGINX_FILE` |

Important rule:

> These variables point to **temporary files**, not workspace files.

Example at runtime:
```text
DEPLOY_SH=/tmp/jenkins123/config456.tmp
```

---

## 5. Correct SCP Commands (CRITICAL)

You must SCP **the variables**, not the original filenames.

```bash
set -e

APP_DIR=/opt/web-demo-frontend
SERVER_USER=deploy
SERVER_IP=1.14.45.120

# ensure target directory exists
ssh ${SERVER_USER}@${SERVER_IP} "mkdir -p ${APP_DIR}"

# copy frontend build output
scp -r dist   ${SERVER_USER}@${SERVER_IP}:${APP_DIR}/

# copy managed config files
scp "$DEPLOY_SH"   ${SERVER_USER}@${SERVER_IP}:${APP_DIR}/deploy.sh

scp "$DOCKER_FILE"   ${SERVER_USER}@${SERVER_IP}:${APP_DIR}/Dockerfile

scp "$NGINX_FILE"   ${SERVER_USER}@${SERVER_IP}:${APP_DIR}/nginx.conf
```

This ensures:
- correct filenames on the server
- no dependency on Jenkins temp paths

---

## 6. Why Dockerfile Is Transferred (Not Preinstalled)

Reasons:
- Dockerfile is **deployment configuration**
- It may change independently of code
- Config File Provider allows centralized management
- No need to rebuild Jenkins image

Dockerfile role:
- Uses `nginx` image
- Copies `dist/` into container
- Starts Nginx to serve frontend

---

## 7. deploy.sh: Purpose and Responsibilities

`deploy.sh` runs **on the remote server**, not in Jenkins.

Typical responsibilities:
- stop old container
- remove old container
- build new Docker image
- run new container

Example structure:

```bash
#!/usr/bin/env bash
set -e

APP_NAME=web-demo-frontend
IMAGE_NAME=web-demo-frontend:latest
NETWORK_NAME=web-demo-net

docker stop $APP_NAME || true
docker rm $APP_NAME || true

docker build -t $IMAGE_NAME .

docker run -d \
  --name $APP_NAME \
  --network $NETWORK_NAME \
  -p 80:80 \
  --restart unless-stopped \
  $IMAGE_NAME
```

---

## 8. Executing deploy.sh via SSH (Heredoc)

After SCP, Jenkins triggers deployment:

```bash
ssh ${SERVER_USER}@${SERVER_IP} << EOF
  set -e
  chmod +x ${APP_DIR}/deploy.sh
  cd ${APP_DIR}
  ./deploy.sh
EOF
```

Why this works well:
- single SSH connection
- clear execution order
- easy to debug
- no line continuation needed

---

## 9. Permissions and Why chmod +x Is Required

After SCP:
- files are usually uploaded as `rw-r--r--`
- shell scripts are **not executable**

So this is required:

```bash
chmod +x deploy.sh
```

Otherwise:
```text
permission denied
```

---

## 10. Summary

- `dist/` is the **only frontend artifact**
- SCP transfers artifacts and config safely
- Dockerfile defines runtime image
- deploy.sh controls container lifecycle
- Jenkins orchestrates, server executes

---

## 11. Final Takeaway

> Frontend deployment in Jenkins is a clean separation of build (Jenkins) and run (server), using SCP for delivery and Docker for runtime isolation.
