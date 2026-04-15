# Jenkins SSH & GitHub Credentials Setup Guide

This document summarizes **how to correctly set up SSH credentials and GitHub credentials in Jenkins**,  
with clear explanations of **when to use each type**, **common mistakes**, and **best practices**.

---

## 1. Why Credentials Matter in Jenkins

Jenkins needs credentials to:
- Pull code from GitHub
- Push artifacts or tags
- SSH into remote servers for deployment

**Credentials are NOT stored in jobs**.  
They are stored centrally and referenced by ID.

---

## 2. Jenkins Credentials System (Core Concepts)

### Credential Components

| Concept | Meaning |
|------|-------|
| Type | What kind of authentication |
| Scope | Where it can be used |
| ID | Unique identifier (most important) |
| Description | Human-readable note |

📌 **Jobs always reference credentials by ID**

---

## 3. Setting Up SSH Credentials (for Server Deployment)

### Use Case
- Deploy JARs via `scp`
- Run commands via `ssh`
- Connect to Linux servers

---

### 3.1 Recommended Type: SSH Username with Private Key

#### Steps

1. Jenkins UI → **Manage Jenkins**
2. **Credentials**
3. (Global) → **Add Credentials**

Set:

- **Kind**: SSH Username with private key
- **Scope**: Global
- **Username**: e.g. `deploy`
- **Private Key**:  
  - ✔ Enter directly  
  - or ✔ From file
- **ID**: `backend-server-ssh`
- **Description**: SSH access to backend server

Save.

---

### 3.2 Prepare Remote Server (One-time)

On the target server:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Add Jenkins public key to:

```bash
~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

---

### 3.3 Using SSH Credential in Jenkins Job

#### Freestyle Job (Execute shell)

```bash
scp target/app.jar deploy@server-ip:/opt/app/app.jar
```

Jenkins automatically injects the SSH key.

---

#### Pipeline Example

```groovy
sshagent(credentials: ['backend-server-ssh']) {
    sh 'scp target/app.jar deploy@server-ip:/opt/app/app.jar'
}
```

---

## 4. Setting Up GitHub Credentials

There are **two correct ways**, depending on your auth method.

---

## 5. GitHub via HTTPS (Recommended for Beginners)

### Credential Type
**Username with password**

📌 GitHub password = **Personal Access Token (PAT)**

---

### 5.1 Create GitHub Token

GitHub → Settings → Developer settings → Personal access tokens

Permissions:
- `repo`
- `read:packages` (if needed)

---

### 5.2 Add GitHub HTTPS Credential

Jenkins → Credentials → Add:

- **Kind**: Username with password
- **Username**: GitHub username
- **Password**: GitHub PAT
- **ID**: `github-https`
- **Description**: GitHub HTTPS access

---

### 5.3 Use in Jenkins Job

#### Freestyle SCM
- Repository URL: `https://github.com/your-org/your-repo.git`
- Credentials: `github-https`

---

## 6. GitHub via SSH (Advanced / Preferred)

### When to use
- No tokens
- Long-term automation
- Better security

---

### 6.1 Generate SSH Key for Jenkins

Inside Jenkins container or host:

```bash
ssh-keygen -t ed25519 -C "jenkins@ci"
```

---

### 6.2 Add Public Key to GitHub

GitHub → Settings → SSH and GPG keys → Add new key

---

### 6.3 Add SSH Credential to Jenkins

- **Kind**: SSH Username with private key
- **Username**: `git`
- **Private Key**: Jenkins private key
- **ID**: `github-ssh`

---

### 6.4 Use SSH Repo URL

```text
git@github.com:org/repo.git
```

Select credential: `github-ssh`

---

## 7. Common Mistakes (AVOID THESE)

❌ Put secrets in Jenkinsfile  
❌ Hardcode passwords in shell  
❌ Commit keys to Git  
❌ Reuse root SSH keys  
❌ Use `--password` in Git commands  

---

## 8. Best Practices Summary

- Use **IDs**, not inline secrets
- Prefer SSH for servers
- Prefer HTTPS + PAT or SSH for GitHub
- Rotate tokens periodically
- Keep Jenkins home on volume

---

## 9. Mental Model (IMPORTANT)

```text
Credential store = vault
Credential ID    = reference
Job              = user
```

Jobs **never know the secret**, only the ID.

---

## 10. One-Sentence Takeaway

> Jenkins credentials are centrally managed, securely stored, and referenced by ID — never hardcoded.

---

**End of document**
