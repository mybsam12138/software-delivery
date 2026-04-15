# Jenkins NodeJS Plugin Summary and Tooling Strategy

This document summarizes **how Jenkins NodeJS Plugin works**, and explains **why using Jenkins plugins is often more convenient than rebuilding the Jenkins Docker image**, based on the experience of installing Maven and Node.js directly into the image earlier.

---

## 1. Background Context

Earlier, the Jenkins Docker image was modified to:

- install Maven via `apt-get`
- install Node.js and npm via NodeSource
- rebuild and rerun the Jenkins container

This approach **works**, but it introduces extra operational cost.

Later, Jenkins **NodeJS Plugin** was introduced, which provides a cleaner and more flexible solution.

---

## 2. What the Jenkins NodeJS Plugin Is

The **NodeJS Plugin** is an official Jenkins plugin that allows Jenkins to:

- download and manage Node.js versions
- inject `node`, `npm`, and `npx` into build jobs
- avoid installing Node.js into the Jenkins Docker image

Node.js is installed under:

```text
/var/jenkins_home/tools/
```

and added to `PATH` only for jobs that enable it.

---

## 3. How the NodeJS Plugin Works

### 3.1 Installation

```text
Manage Jenkins
 → Plugins
 → Available plugins
 → NodeJS Plugin
```

After installation, Node.js versions are configured globally.

---

### 3.2 Global Tool Configuration

```text
Manage Jenkins
 → Global Tool Configuration
 → NodeJS
```

Example configuration:

- Name: `Node20`
- Install automatically: ✅
- Version: `NodeJS 20.x`

Jenkins downloads and caches Node.js automatically.

---

### 3.3 Job-Level Usage (Freestyle)

In a Freestyle job:

```text
Build Environment
 → Provide Node & npm bin/ folder to PATH
 → Select: Node20
```

After this, build steps can directly run:

```bash
node -v
npm -v
npm install
npm run build
```

---

## 4. Why NodeJS Plugin Is More Convenient Than Rebuilding Images

### 4.1 Rebuilding Jenkins Image (Earlier Approach)

Pros:
- Works reliably
- Tools always available

Cons:
- Requires Dockerfile changes
- Requires image rebuild
- Requires container restart
- Tool versions are tightly coupled to Jenkins image
- Harder to manage multiple Node versions

---

### 4.2 NodeJS Plugin Approach (Better for Most Cases)

Pros:
- No Jenkins image rebuild
- No container restart
- Tool versions managed in UI
- Easy Node.js version upgrade/downgrade
- Multiple Node versions supported
- Tools persist via `/var/jenkins_home`

Cons:
- Slight Jenkins-level dependency
- Requires plugin installation (one-time)

---

## 5. Maven Comparison (Important Insight)

Earlier, Maven was installed directly into the Jenkins image.

However, Jenkins also provides:

- **Maven Integration Plugin**
- **Global Maven Tool Configuration**

Which means Maven can be:

- installed automatically by Jenkins
- versioned per job
- managed without rebuilding the Jenkins image

This shows a clear pattern:

> **Jenkins plugins are usually a better place to manage build tools than Docker images.**

---

## 6. Recommended Tooling Strategy

### 6.1 What Should Be in the Jenkins Image

Keep the Jenkins image **minimal**:

- Jenkins LTS
- OS dependencies only
- No language-specific build tools

---

### 6.2 What Should Be Managed by Jenkins Plugins

Use plugins for:

- Node.js / npm → NodeJS Plugin
- Maven → Maven Tool Configuration
- settings.xml → Config File Provider

This gives:
- flexibility
- faster iteration
- less image maintenance

---

## 7. When Rebuilding the Jenkins Image Still Makes Sense

Rebuilding the Jenkins image is reasonable when:

- Jenkins runs in an isolated environment
- No plugin installation allowed
- Tools must be pinned at OS level
- Jenkins agents are immutable

But for most application CI pipelines, plugins are preferred.

---

## 8. Key Takeaways

- Jenkins NodeJS Plugin provides Node.js and npm cleanly
- Tools are injected per job, not globally
- No Jenkins image rebuild required
- Easier upgrades and maintenance
- More flexible than baking tools into the image

---

## 9. Final Takeaway

> Using Jenkins plugins (NodeJS, Maven) to manage build tools is more flexible and maintainable than rebuilding the Jenkins Docker image, especially when `/var/jenkins_home` is persisted.

---

**End of document**
