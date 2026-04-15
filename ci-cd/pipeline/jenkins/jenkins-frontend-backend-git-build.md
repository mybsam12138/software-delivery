# Jenkins Frontend & Backend Git Checkout and Build Process

This document summarizes **how Jenkins checks out code and builds both frontend and backend projects**, focusing only on the **Git checkout and build stages** (no deployment).

It is written for **Freestyle Jenkins jobs**, but the process applies equally to Pipelines.

---

## 1. Overall Mental Model

```text
Jenkins Job
  ↓
Git Checkout
  ↓
Build (Frontend / Backend)
  ↓
Artifacts ready for deployment
```

Key principle:

> **Jenkins is responsible for source checkout and build.  
> Servers are responsible only for running Docker containers.**

---

## 2. Git Checkout in Jenkins

### 2.1 Source Code Management Configuration

In a Freestyle job:

```text
Job → Configure → Source Code Management → Git
```

You configure:

- Repository URL (HTTP or SSH)
- Credentials (GitHub / GitLab / private repo)
- Branch (e.g. `main`, `master`, `develop`)

Example:

```text
Repository URL: git@github.com:your-org/web-demo-backend.git
Credentials: github-ssh-key
Branch: */main
```

After checkout, Jenkins creates a workspace:

```text
/var/jenkins_home/workspace/JOB_NAME/
```

---

### 2.2 Workspace Structure After Checkout

#### Backend job example

```text
workspace/
├── pom.xml
├── src/
└── target/        (created after build)
```

#### Frontend job example

```text
workspace/
├── package.json
├── package-lock.json
├── src/
└── dist/          (created after build)
```

---

## 3. Backend Build Process (Spring Boot)

### 3.1 Backend Build Command

In **Execute shell**:

```bash
set -e

mvn clean package
```

What happens:

- Jenkins uses Maven (plugin or image-installed)
- Dependencies are resolved
- Code is compiled and tested
- A runnable JAR is produced

---

### 3.2 Backend Build Output

After build:

```text
target/
└── web-demo-backend-<version>.jar
```

This JAR is the **only backend build artifact** needed for deployment.

---

### 3.3 Key Backend Build Rules

- ❌ Do not build backend on the server
- ❌ Do not SCP the whole `target/` directory
- ✅ Only the final JAR is deployed
- ✅ Version can be fixed or renamed to `app.jar`

---

## 4. Frontend Build Process (Node.js)

### 4.1 Node Environment in Jenkins

Node.js and npm are provided by:

- **Jenkins NodeJS Plugin**
- Injected into PATH at job runtime

No Node.js installation is required in the Jenkins image.

---

### 4.2 Frontend Build Commands

In **Execute shell**:

```bash
set -e

npm install
npm run build
```

(or `npm ci` if strict reproducibility is required)

What happens:

- Dependencies are resolved
- Frontend code is compiled
- Static assets are generated

---

### 4.3 Frontend Build Output

After build:

```text
dist/
├── index.html
├── assets/
└── ...
```

The `dist/` directory is the **only frontend build artifact**.

---

### 4.4 Key Frontend Build Rules

- ❌ Do not run npm on the server
- ❌ Do not deploy `node_modules`
- ✅ Only `dist/` is deployed
- ✅ Frontend build always happens in Jenkins

---

## 5. Frontend vs Backend Build Comparison

| Aspect | Frontend | Backend |
|------|---------|---------|
| Build tool | npm | Maven |
| Runtime | Nginx | JVM |
| Build output | `dist/` | `*.jar` |
| Built in Jenkins | Yes | Yes |
| Built on server | No | No |

---

## 6. Common Build Mistakes to Avoid

- Running builds on the deployment server
- Mixing build and deploy logic
- Checking in build artifacts to Git
- Assuming workspace is always clean
- Forgetting to stop build on errors (`set -e`)

---

## 7. Recommended Jenkins Job Structure

```text
Job A: web-demo-backend-build
  1. Git checkout
  2. mvn clean package

Job B: web-demo-frontend-build
  1. Git checkout
  2. npm install
  3. npm run build
```

Each job produces **clean, deployable artifacts**.

---

## 8. Final Takeaway

> Jenkins should handle Git checkout and builds for both frontend and backend, producing clean artifacts (`dist/` and `.jar`) that are later deployed by Docker-based processes.

---

**End of build process summary**
