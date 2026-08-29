# Spring Petclinic – Docker, CI/CD & Kubernetes

This project demonstrates containerization, Continuous Integration (CI), and Continuous Deployment (CD) for the Spring Petclinic application using Docker, GitHub Actions, Docker Hub, and Kubernetes (Kind).

## Architecture

```text
                         Developer
                            |
                            | Push/Pull request to main
                            v
                  +---------------------+
                  | GitHub Actions - CI |
                  +---------------------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
           Build          Tests      Docker Build
                                            |
                                            v
                                      Docker Hub
                                            |
                                            v
                                CI workflow succeeds
                                            |
                                            v
                  +-----------------------------+
                  | GitHub Actions - CD         |
                  | Self-hosted Runner          |
                  +-----------------------------+
                                |
                                v
                         Kind Kubernetes
                                |
                  +-------------+-------------+
                  |                           |
                  v                           v
          Petclinic Deployment        MySQL StatefulSet
                  |                           |
                  v                           v
          Petclinic Service             Persistent PVC
```

## Technologies

* Java 17
* Spring Boot
* Maven
* Docker
* Docker Compose
* Docker Hub
* GitHub Actions
* Kubernetes
* Kind
* MySQL 8.0

## Project Structure

```text
spring-petclinic/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
│
├── k8s/
│   ├── mysql.yml
│   └── petclinic.yml
│
├── Dockerfile
├── docker-compose.yml
├── pom.xml
└── mvnw
```

## Docker

The application uses a multi-stage Docker build.

### Build stage

The Dockerfile:

1. Copies the Maven wrapper and `pom.xml`
2. Downloads Maven dependencies
3. Copies the source code
4. Builds the application

### Runtime stage

A smaller Java Runtime image is used to run the generated JAR.

```text
Maven/JDK image
      |
      | Build
      v
   Spring Boot JAR
      |
      v
Java JRE runtime image
```

## Local Docker Compose

The application can be started locally with:

```bash
docker compose up --build
```

Services:

```text
Application: http://localhost:8080
MySQL:       localhost:3306
```

Run in detached mode:

```bash
docker compose up -d
```

Check containers:

```bash
docker ps
```

Stop the application:

```bash
docker compose down
```

## Continuous Integration – CI

CI is implemented using GitHub Actions.

Workflow:

```text
Checkout source code
        |
        v
Setup Java 17
        |
        v
Build application
        |
        v
Run tests
        |
        v
Build Docker image
        |
        v
Login to Docker Hub
        |
        v
Push Docker image
```

The CI workflow is located at:

```text
.github/workflows/ci.yml
```

### Tests

The PostgreSQL integration test is excluded because this project uses MySQL.

The CI test command is:

```bash
./mvnw clean test -Dtest='!PostgresIntegrationTests'
```

The test suite was verified successfully with:

```text
Tests run: 74
Failures: 0
Errors: 0
Skipped: 0
BUILD SUCCESS
```

### Docker Hub

The generated image is published to:

```text
mohamedwanes126/spring-petclinic
```

Images are tagged with:

```text
latest
<Git commit SHA>
```

Docker Hub:

https://hub.docker.com/r/mohamedwanes126/spring-petclinic

## Continuous Deployment – CD

CD is implemented using GitHub Actions and a self-hosted runner.

The self-hosted runner runs on the same machine as the local Kind Kubernetes cluster.

Workflow:

```text
Successful CI
      |
      v
Self-hosted GitHub Actions Runner
      |
      v
Create/Update Kubernetes Secret
      |
      v
Apply Kubernetes manifests
      |
      v
Verify MySQL rollout
      |
      v
Verify Petclinic rollout
      |
      v
Check Pods / PVC / Services
      |
      v
Check application accessibility
```

The CD workflow is located at:

```text
.github/workflows/cd.yml
```

## Kubernetes

The Kubernetes cluster is created using Kind.

Check the cluster:

```bash
kubectl get nodes
```

Expected:

```text
petclinic-control-plane   Ready
```

### Kubernetes Resources

#### MySQL

MySQL is deployed using a StatefulSet because database workloads require persistent storage.

Resources:

```text
StatefulSet
Service
PersistentVolumeClaim
ConfigMap
Secret
```

#### Petclinic

The application is deployed using a Deployment.

Resources:

```text
Deployment
Service
ConfigMap
Secret
```

### Kubernetes Manifests

All Kubernetes manifests are stored in:

```text
k8s/
```

Apply MySQL:

```bash
kubectl apply -f k8s/mysql.yml
```

Apply Petclinic:

```bash
kubectl apply -f k8s/petclinic.yml
```

Check resources:

```bash
kubectl get pods
kubectl get pvc
kubectl get svc
```

Verify rollout:

```bash
kubectl rollout status statefulset/mysql
kubectl rollout status deployment/petclinic
```

## Kubernetes Secrets

Sensitive values are not stored in the Kubernetes manifests committed to Git.

GitHub Actions repository secrets are used for:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
MYSQL_ROOT_PASSWORD
MYSQL_PASSWORD
```

The CD workflow creates or updates the Kubernetes Secret during deployment.

## Accessing the Application

The Petclinic service is exposed through a Kubernetes NodePort.

For local verification, the service can be accessed using:

```bash
kubectl port-forward service/petclinic 8080:8080
```

Then open:

```text
http://localhost:8080
```

The deployment was verified successfully by requesting the application endpoint and receiving the Spring Petclinic HTML response.

## CI/CD Verification

### CI

* GitHub Actions workflow completed successfully
* Application build succeeded
* 74 tests passed
* Docker image built successfully
* Docker image pushed to Docker Hub

<img width="1916" height="905" alt="image" src="https://github.com/user-attachments/assets/47dca38b-f85e-4aa8-994d-b8ac4ee8dceb" />

<img width="1900" height="907" alt="image" src="https://github.com/user-attachments/assets/5af9eabd-1531-4744-b18c-eea5cf2d906c" />

### CD

* Self-hosted runner connected successfully
* Kind Kubernetes cluster is available
* MySQL StatefulSet deployed successfully
* PersistentVolumeClaim bound successfully
* Petclinic Deployment rolled out successfully
* Kubernetes services created successfully
* Application accessibility verified successfully

<img width="1917" height="911" alt="image" src="https://github.com/user-attachments/assets/75b7afa8-842e-4b64-8f14-91310b4623ed" />

<img width="1200" height="410" alt="image" src="https://github.com/user-attachments/assets/6565e94f-1e54-48e9-9cd4-dde7a5cdf1b8" />

<img width="1916" height="962" alt="image" src="https://github.com/user-attachments/assets/5348a6fb-bb08-4e12-a45f-0750fe068ceb" />

<img width="1917" height="895" alt="image" src="https://github.com/user-attachments/assets/b53de62d-a90d-4b20-aa91-45c8981193b6" />


## Useful Commands

### Git

```bash
git status
git log --oneline
git push origin main
```

### Docker

```bash
docker ps
docker images
docker compose up -d
docker compose down
```

### Kubernetes

```bash
kubectl get nodes
kubectl get pods
kubectl get svc
kubectl get pvc
kubectl get statefulset
kubectl get deployment
kubectl rollout status deployment/petclinic
```

## Repository

GitHub:

https://github.com/MohamedWanes/spring-petclinic

Docker Hub:

https://hub.docker.com/r/mohamedwanes126/spring-petclinic
