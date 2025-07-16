# Vprofile App: CI/CD Pipeline

This README documents the **Continuous Integration (CI)** and **Continuous Deployment (CD)** pipeline for the **Vprofile application**.  
The pipeline is implemented in **Jenkins** and automates the process from code commit ➡️ artifact validation ➡️ containerization ➡️ staging deployment ➡️ production deployment on ECS.

---

##  Overview

**Goals of this pipeline:**
- Automate build and test of the Vprofile Java application  
- Enforce coding standards and perform static analysis  
- Manage build artifacts in Nexus  
- Build and push Docker images to ECR  
- Deploy automatically to ECS **staging** and then **production**  
- Notify developers on every stage  

---

## 📊 Visual Flow Diagram

![CI Pipeline Diagram](ci/cd-vprofile-diagram.png) 
---

## Tools and Services Used

| Purpose                | Tool/Service             |
|------------------------|--------------------------|
| CI/CD Orchestration    | Jenkins                  |
| Version Control        | Git                      |
| Build Automation       | Maven                    |
| Code Style Analysis    | Checkstyle               |
| Static Code Analysis   | SonarQube                |
| Artifact Repository    | Nexus Repository Manager |
| Containerization       | Docker                   |
| Image Registry         | AWS ECR                  |
| Orchestration          | AWS ECS                  |
| Notifications          | Slack                    |

---

## CI Pipeline Stages

### Build
```bash
mvn -s settings.xml -DskipTests install
```

### Test
```bash
mvn -s settings.xml test
```

### Checkstyle
```bash
mvn -s settings.xml checkstyle:checkstyle
```

### SonarQube Analysis
```bash
sonar-scanner   -Dsonar.projectKey=vprofile   -Dsonar.sources=src/   -Dsonar.java.binaries=target/
```

### Upload Artifact to Nexus
```groovy
artifactId: 'vproapp'
file: 'target/vprofile-v2.war'
type: 'war'
repository: 'vprofile-release'
```

---

## CD Pipeline Stages

### Build App Image
```groovy
dockerImage = docker.build(appRegisrty + ":$BUILD_NUMBER", "-f ./Docker-files/app/Dockerfile .")
```

### Upload App Image
```groovy
docker.withRegistry(vprofileRegistry, registryCredentinal) {
    dockerImage.push("$BUILD_NUMBER")
    dockerImage.push("latest")
}
```

### Deploy to Staging ECS
```bash
aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment
```

---

### Deploy to Production ECS
Once staging tests succeed, the same image is deployed to production:
```bash
aws ecs update-service --cluster ${prodCluster} --service ${prodService} --force-new-deployment
```
---

## 🧩 Notifications

- Slack messages are sent after each stage:
  - Build status
  - Deployment status (staging & production)
  - Jenkins job link

---

## Security

- All credentials (Docker, AWS) are stored in **Jenkins Credentials Manager**.
- They are injected at runtime as environment variables.
- IAM roles use least-privilege access.

