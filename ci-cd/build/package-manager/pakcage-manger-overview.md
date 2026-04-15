# Package Managers Overview

## 1. What is a Package Manager?

A **package manager** is a tool that helps you:

- Install software
- Update software
- Remove software
- Manage dependencies (automatically install required libraries)

---

## 2. Linux Package Managers

### 2.1 CentOS / RHEL (YUM / DNF)

#### 🔹 What it is
- **YUM (Yellowdog Updater Modified)** — older versions
- **DNF (Dandified YUM)** — newer replacement (CentOS 8+)

#### 🔹 Purpose
- Manage system-level software packages (usually `.rpm` files)

#### 🔹 Common Commands

```bash
# Install package
sudo yum install nginx

# Update system
sudo yum update

# Remove package
sudo yum remove nginx

# Search package
yum search nginx
```

#### 🔹 Key Features
- Handles `.rpm` packages
- Automatically resolves dependencies
- Uses remote repositories

---

### 2.2 Ubuntu / Debian (APT)

#### 🔹 What it is
- **APT (Advanced Package Tool)**

#### 🔹 Purpose
- Manage system packages (`.deb` files)

#### 🔹 Common Commands

```bash
# Update package list
sudo apt update

# Install package
sudo apt install nginx

# Upgrade packages
sudo apt upgrade

# Remove package
sudo apt remove nginx
```

#### 🔹 Key Features
- Works with `.deb` packages
- Very fast and widely used
- Strong repository ecosystem

---

## 3. Language-Level Package Managers

### 3.1 npm (Node Package Manager)

#### 🔹 What it is
- Package manager for **JavaScript / Node.js**

#### 🔹 Purpose
- Manage project dependencies (libraries, frameworks)

#### 🔹 Example Use Case
- Installing frontend libraries like React, Vue, Axios

#### 🔹 Common Commands

```bash
# Initialize project
npm init

# Install package
npm install axios

# Install globally
npm install -g typescript

# Install all dependencies
npm install

# Remove package
npm uninstall axios
```

#### 🔹 Key Files

- `package.json` → defines dependencies
- `node_modules/` → installed packages

#### 🔹 Key Features
- Huge ecosystem (millions of packages)
- Version control of dependencies
- Local (project) + global installs

---

### 3.2 Maven

#### 🔹 What it is
- Build and dependency management tool for **Java**

#### 🔹 Purpose
- Manage Java project dependencies
- Build, package, and run Java applications

#### 🔹 Example Use Case
- Spring Boot backend projects

#### 🔹 Common Commands

```bash
# Compile project
mvn compile

# Package project (generate jar)
mvn package

# Run tests
mvn test

# Install to local repository
mvn install
```

#### 🔹 Key File

- `pom.xml` → defines:
    - dependencies
    - plugins
    - project configuration

#### 🔹 Example Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.0.0</version>
</dependency>
```

#### 🔹 Key Features
- Strong dependency management
- Lifecycle-based build system
- Uses central repository (Maven Central)

---

## 4. Key Differences (High-Level)

| Type              | Tool        | Scope            | Language        |
|------------------|------------|------------------|-----------------|
| System-level     | yum / dnf  | OS packages      | Linux (CentOS)  |
| System-level     | apt        | OS packages      | Linux (Ubuntu)  |
| Language-level   | npm        | Project deps     | JavaScript      |
| Language-level   | Maven      | Project deps + build | Java       |

---

## 5. Core Concept Summary

### 🔹 System Package Managers (apt, yum)
- Install **software for the OS**
- Example: nginx, docker, git

### 🔹 Language Package Managers (npm, maven)
- Install **libraries for your code**
- Example:
    - npm → axios, react
    - maven → spring-boot, jackson

---

## 6. Simple Mental Model

- **apt / yum** → install tools on your machine  
  👉 "I want nginx installed on my server"

- **npm / maven** → install libraries for your project  
  👉 "My project needs axios / spring"

---

## 7. Typical Workflow

### Backend (Java)

```bash
# Install Java (system level)
sudo apt install openjdk-17

# Build project (maven)
mvn package
```

---

### Frontend (Node.js)

```bash
# Install Node.js (system level)
sudo apt install nodejs

# Install dependencies
npm install

# Run project
npm run dev
```

---

## 8. Final Summary

- Package managers solve **dependency + installation problems**
- There are **two layers**:
    - OS level → apt / yum
    - Project level → npm / maven
- Modern development always uses both layers together

---

## 9. One Sentence Summary

> System package managers install software for your machine, while language package managers install dependencies for your code.