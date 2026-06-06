# Gradle Summary

## 1. What is Gradle?

Gradle is a build automation and dependency management tool primarily used for Java, Spring Boot, Kotlin, and Android projects.

Its responsibilities include:

- Dependency management
- Source code compilation
- Test execution
- Packaging (JAR/WAR)
- Application execution
- CI/CD integration
- Custom build automation

Think of Gradle as the tool that converts source code into a deployable application.

---

## 2. Why Use Gradle?

Common use cases:

- Download libraries from Maven Central
- Compile Java code
- Run unit tests
- Build JAR/WAR files
- Run Spring Boot applications
- Manage multi-module projects
- Automate repetitive tasks

Typical commands:

```bash
./gradlew clean
./gradlew test
./gradlew build
./gradlew bootRun
```

---

## 3. Core Gradle Files

### build.gradle

Main build configuration file.

```groovy
plugins {
    id 'java'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

### settings.gradle

Defines project name and modules.

```groovy
rootProject.name = 'my-app'
```

### gradlew

Gradle Wrapper.

Allows developers to use a specific Gradle version without manually installing Gradle.

```bash
./gradlew build
```

---

## 4. Core Gradle Example

### Simple Spring Boot Project

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.0'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.example'
version = '1.0.0'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

Run application:

```bash
./gradlew bootRun
```

Build application:

```bash
./gradlew build
```

Generated artifact:

```text
build/libs/app.jar
```

---

## 5. Gradle Dependency Scopes

| Gradle | Purpose |
|----------|----------|
| implementation | Normal compile dependency |
| api | Exposed to dependent modules |
| compileOnly | Compile only, not packaged |
| runtimeOnly | Needed only at runtime |
| testImplementation | Test dependency |

Example:

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    runtimeOnly 'com.mysql:mysql-connector-j'

    testImplementation 'org.junit.jupiter:junit-jupiter'
}
```

---

## 6. Tasks in Gradle

Everything in Gradle is a Task.

View available tasks:

```bash
./gradlew tasks
```

Custom task:

```groovy
task hello {
    doLast {
        println "Hello Gradle"
    }
}
```

Run:

```bash
./gradlew hello
```

---

## 7. Multi-Module Project Example

Project structure:

```text
project
├── common
├── user-service
├── order-service
└── gateway
```

settings.gradle:

```groovy
include 'common'
include 'user-service'
include 'order-service'
include 'gateway'
```

Reference another module:

```groovy
dependencies {
    implementation project(':common')
}
```

---

# Gradle vs Maven

## High-Level Comparison

| Feature | Gradle | Maven |
|----------|----------|----------|
| Configuration | Groovy/Kotlin DSL | XML |
| Readability | Concise | Verbose |
| Performance | Faster (incremental build, caching) | Generally slower |
| Flexibility | Highly customizable | Convention-based |
| Learning Curve | Medium | Easy |
| Custom Tasks | Very powerful | More limited |
| Multi-module Support | Excellent | Excellent |
| Spring Boot Support | Excellent | Excellent |
| Android Development | Standard choice | Rarely used |

---

## Dependency Example

### Maven

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### Gradle

```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
```

Gradle is significantly more concise.

---

## Build Command Comparison

| Action | Maven | Gradle |
|----------|----------|----------|
| Clean | mvn clean | ./gradlew clean |
| Test | mvn test | ./gradlew test |
| Package | mvn package | ./gradlew build |
| Run Spring Boot | mvn spring-boot:run | ./gradlew bootRun |

---

# Interview Summary

For a Senior Java Engineer, focus on:

1. What Gradle is
2. Gradle Wrapper (gradlew)
3. build.gradle structure
4. Dependency scopes
5. Common commands
6. Custom tasks
7. Multi-module projects
8. Gradle vs Maven differences

If you understand these topics, you can comfortably work on most modern Spring Boot projects using Gradle.
