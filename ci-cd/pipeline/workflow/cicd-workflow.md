# CI/CD Workflow using Jenkins and Kubernetes

## 1. Overview

CI/CD (Continuous Integration / Continuous Delivery) automates the process from **code commit → build → test → image build → deployment**.

Typical workflow:

Developer Push Code  
↓  
Git Repository (GitHub / GitLab)  
↓  
Jenkins Pipeline Triggered  
↓  
Build + Static Analysis  
↓  
Unit Tests + Integration Tests  
↓  
Build Docker Image  
↓  
Push Image to Registry  
↓  
Deploy to Kubernetes  
↓  
Run Container in Cluster

Goals:

- Automate build and deployment
- Ensure code quality
- Reduce manual operations
- Enable fast and safe releases

---

# 2. CI/CD Architecture

Typical components in the pipeline.

| Component | Purpose |
|---|---|
| Git Repository | Source code management |
| Jenkins | CI/CD pipeline orchestrator |
| Maven / Gradle | Build tool |
| SonarQube | Static code analysis |
| Docker | Container packaging |
| Container Registry | Store Docker images |
| Kubernetes | Container orchestration |

Architecture Flow:

Developer  
→ Git Push  
→ Git Repository  
→ Jenkins Pipeline  
→ Build + Test  
→ Docker Image  
→ Image Registry  
→ Kubernetes Deployment  
→ Running Pods

---

# 3. CI/CD Pipeline Stages

## Stage 1 — Pull Source Code

Jenkins pulls the source code from Git.

Example:

```
git clone https://github.com/project/app.git
```

Triggers:

- Git webhook
- scheduled pipeline
- manual build

---

# Stage 2 — Build Project

Compile the project using Maven or Gradle.

Example:

```
mvn clean compile
```

Build artifact:

```
target/app.jar
```

Purpose:

- compile code
- resolve dependencies
- produce build artifact

---

# Stage 3 — Static Code Analysis

Check code quality and security.

Common tools:

- SonarQube
- Checkstyle
- SpotBugs
- PMD

Example:

```
mvn sonar:sonar
```

Checks include:

- code smells
- duplicated code
- security vulnerabilities
- complexity

If quality gate fails → pipeline stops.

---

# Stage 4 — Unit Tests

Run automated tests for individual components.

Example:

```
mvn test
```

Purpose:

- validate business logic
- test service layer
- test utility functions

Examples:

- Service layer tests
- Repository tests
- Utility tests

If tests fail → pipeline stops.

---

# Stage 5 — Integration Tests

Test interaction between multiple components.

Example:

```
mvn verify
```

Typical integration tests include:

- database integration
- REST API tests
- service communication
- container-based testing

Tools:

- SpringBootTest
- TestContainers
- Mock external services

---

# Stage 6 — Build Docker Image

Package the application into a container image.

Example Dockerfile:

```
FROM openjdk:21-jdk

COPY target/app.jar app.jar

ENTRYPOINT ["java","-jar","/app.jar"]
```

Build image:

```
docker build -t app:1.0 .
```

Purpose:

- package application
- ensure consistent runtime environment

---

# Stage 7 — Push Image to Registry

Push the image to a container registry.

Common registries:

- DockerHub
- Harbor
- AWS ECR
- GitHub Packages

Example:

```
docker tag app:1.0 registry/app:1.0
docker push registry/app:1.0
```

The registry stores versioned images for deployment.

---

# Stage 8 — Deploy to Kubernetes

Update Kubernetes deployment to use the new image.

Example:

```
kubectl set image deployment/app app=registry/app:1.0
```

Kubernetes performs a **rolling update**.

Example Deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: registry/app:1.0
```

Features:

- automatic rolling updates
- health checks
- scaling

---

# Stage 9 — Run Application in Pods

Kubernetes schedules containers to nodes.

Example structure:

Node  
├─ Pod  
│   └─ Container (Spring Boot App)  
├─ Pod  
│   └─ Container  
├─ Pod  
│   └─ Container

Capabilities:

- load balancing
- auto scaling
- restart failed containers
- service discovery

---

# 4. Jenkins Pipeline Example

Example Jenkinsfile:

```
pipeline {

  agent any

  stages {

    stage('Checkout') {
      steps {
        git 'https://github.com/project/app.git'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Static Analysis') {
      steps {
        sh 'mvn sonar:sonar'
      }
    }

    stage('Unit Test') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Integration Test') {
      steps {
        sh 'mvn verify'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t registry/app:${BUILD_NUMBER} .'
      }
    }

    stage('Push Image') {
      steps {
        sh 'docker push registry/app:${BUILD_NUMBER}'
      }
    }

    stage('Deploy') {
      steps {
        sh 'kubectl set image deployment/app app=registry/app:${BUILD_NUMBER}'
      }
    }

  }

}
```

---

# 5. CI/CD Best Practices

## 1. Keep pipelines simple

Typical pipeline:

```
build
test
image
deploy
```

---

## 2. Fail fast

Stop pipeline immediately if:

- build fails
- tests fail
- security scan fails

---

## 3. Immutable images

Never modify running containers.

Always deploy a **new image version**.

---

## 4. Environment separation

Typical environments:

```
dev
test
uat
prod
```

Deployment flow:

```
dev → test → uat → prod
```

---

## 5. Use Kubernetes rolling updates

Avoid downtime during deployment.

Example configuration:

```
maxUnavailable: 0
maxSurge: 1
```

---

# 6. Full CI/CD Workflow Summary

Complete pipeline:

Developer Push Code  
↓  
Git Repository  
↓  
Jenkins Triggered  
↓  
Pull Code  
↓  
Build Project  
↓  
Static Code Analysis  
↓  
Unit Tests  
↓  
Integration Tests  
↓  
Build Docker Image  
↓  
Push Image to Registry  
↓  
Deploy to Kubernetes  
↓  
Kubernetes Runs Pods

Final result:

- automated builds
- automated testing
- automated deployments
- reliable release process