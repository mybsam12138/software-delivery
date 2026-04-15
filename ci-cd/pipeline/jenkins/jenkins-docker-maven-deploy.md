# Jenkins + Maven Docker Deployment Guide

This guide summarizes how to deploy **Jenkins with Maven using Docker**, based on real-world CI/CD best practices.

---

## 1. Core Design Principles

- **Jenkins container is disposable**
- **Jenkins home must be persistent**
- **Maven configuration should not be baked into images**
- **Build configuration is injected at runtime**

Key idea:

> Image = tools  
> Container runtime = configuration + data

---

## 2. Jenkins Docker Image Choice

Use the official LTS image:

```bash
jenkins/jenkins:lts
```

Why LTS:
- Stable
- Plugin compatible
- Long-term support

---

## 3. Jenkins Home Persistence (CRITICAL)

Always mount `/var/jenkins_home`.

### Named volume (recommended)

```bash
docker volume create jenkins_home
```

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

What is persisted:
- Jobs
- Plugins
- Credentials
- Managed config files
- Build history

If the container is removed, **data remains**.

---

## 4. Installing Maven (Two Approaches)

### Option A (Recommended): Use system Maven inside Jenkins

Install Maven in the Jenkins image:

```dockerfile
FROM jenkins/jenkins:lts

USER root
RUN apt-get update && \
    apt-get install -y maven && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER jenkins
```

Why acceptable:
- Simple
- Works well for small / medium setups

---

### Option B (Advanced): Dockerized Maven (later stage)

Use `maven:3.x` images in pipelines.

Not recommended for beginners.

---

## 5. Maven settings.xml (CORRECT WAY)

### ❌ Do NOT
- Put `settings.xml` in Dockerfile
- Edit `/root/.m2`
- Replace global Maven config

### ✅ DO
Use **Jenkins Config File Provider Plugin**

---

## 6. Config File Provider Setup

1. Install plugin: **Config File Provider**
2. Jenkins UI → Manage Jenkins → Managed files
3. Add:
   - Type: Maven settings.xml
   - ID: `maven-setting-xml`

This file is stored in:

```text
/var/jenkins_home
```

---

## 7. How Jenkins Uses settings.xml Internally

During a build:
1. Jenkins copies the managed file
2. Places it in `/tmp/configXXXX.xml`
3. Exposes path via environment variable

Example:

```bash
MAVEN_SETTINGS=/tmp/config123456.xml
```

The temp file is deleted after the build.

---

## 8. Freestyle Job: Correct Maven Command

### Build Environment
- ✔ Provide configuration files
- File: `maven-setting-xml`
- Variable: `MAVEN_SETTINGS`

### Execute shell

```bash
mvn -s "$MAVEN_SETTINGS" clean package -DskipTests
```

This overrides default Maven settings **only for this build**.

---

## 9. Why Local Build Works but Jenkins Fails (Common Issue)

Reasons:
- Local `.m2` cache is warm
- Jenkins `.m2` cache is empty
- Jenkins mirror config may be too strict
- Old plugin dependencies not fully mirrored

CI always reveals hidden dependency problems.

---

## 10. Maven Mirror Best Practice (China-Friendly)

Avoid:

```xml
<mirrorOf>*</mirrorOf>
```

Use instead:

```xml
<mirrorOf>central</mirrorOf>
```

This allows fallback for plugins and metadata.

---

## 11. Successful Build = Ready to Deploy

After build success:

```text
target/
└── web-demo-backend-*.jar
```

This JAR is the deployment artifact.

---

## 12. Simple Deployment Flow (No Docker for App Yet)

1. Jenkins builds JAR
2. Jenkins copies JAR to server (scp)
3. Server runs:

```bash
nohup java -jar app.jar &
```

This is the correct **first deployment step**.

---

## 13. Key Takeaways

- Jenkins container can be deleted safely
- Jenkins data survives via volume
- Maven config is injected, not installed
- CI must be reproducible and clean
- Build ≠ Deploy

---

## 14. Mental Model (IMPORTANT)

```text
Docker image  → tools
Docker volume → state
Jenkins UI    → configuration
CI build      → temporary execution
```

---

## 15. Next Recommended Steps

- Convert deployment to Docker
- Add systemd or container restart strategy
- Introduce pipeline (Jenkinsfile)
- Cache `.m2` safely
- Upgrade plugins and Java version

---

**End of document**
