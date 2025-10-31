# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DevSecOps CI/CD pipeline demonstration project featuring a Spring Boot boardgame application with comprehensive security scanning, code quality analysis, and automated deployment workflows. The application is a RESTful web service for managing board games with integrated Spring Security authentication.

## Build & Development Commands

### Maven Build
```bash
# Clean and package (skip tests)
mvn clean package -DskipTests=true

# Full build with tests
mvn package

# Run locally
java -jar target/*.jar
```

### Docker
```bash
# Build image (multi-stage build with maven:3.8.3-openjdk-17 and openjdk:17-alpine)
docker build -t boardgame:latest .

# Run with docker-compose
docker-compose up -d

# Access application at http://localhost:8080
```

### Kubernetes Deployment
```bash
# Apply Kubernetes manifests
kubectl apply -f boardgame.yaml

# The deployment creates:
# - Namespace: boardgame
# - Deployment with 2 replicas
# - Service (ClusterIP on port 80)
# - Ingress with TLS
```

### Testing
```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=TestDatabase
mvn test -Dtest=BoardGameController
```

## Architecture & Code Structure

### Application Architecture
This is a Spring Boot 2.5.6 application using:
- **Java 11** (runtime compatibility)
- **Spring Security** with JDBC authentication
- **H2 in-memory database** for development
- **Thymeleaf** for server-side rendering
- **JPA/JDBC** for data persistence
- **JaCoCo** for code coverage reporting

### Key Components

**Controllers** (`src/main/java/com/javaproject/controllers/`):
- `BoardGameController`: REST API endpoints at `/boardgames` for CRUD operations on board games
- `HomeController`: Web UI routes for authentication and user management

**Security** (`src/main/java/com/javaproject/security/`):
- `SecurityConfig`: Spring Security configuration with role-based access control
  - Default users: `bugs` (USER role) and `daffy` (USER, MANAGER roles)
  - BCrypt password encoding
  - JDBC authentication against H2 database
  - Routes: `/user/**` (USER/MANAGER), `/manager/**` (MANAGER only), `/secured/**` (authenticated)
- `LoggingAccessDeniedHandler`: Custom access denied handler

**Data Layer** (`src/main/java/com/javaproject/database/`):
- `DatabaseAccess`: Data access layer for board game operations

**Domain Models** (`src/main/java/com/javaproject/beans/`):
- `BoardGame`: Board game entity
- `Review`: Review entity
- `ErrorMessage`: Error response model

### Database Configuration
- Uses H2 in-memory database (`jdbc:h2:mem:testdb`)
- Schema defined in `src/main/resources/schema.sql`
- H2 console enabled for development access

## CI/CD Pipeline Architecture

### Pipeline Stages (GitHub Actions & GitLab CI)
1. **Checkout**: Clone repository
2. **Setup**: Install dependencies (Maven, jq)
3. **Build**: Maven package with artifact upload
4. **Security Scan**: Trivy filesystem scan for vulnerabilities
5. **Code Quality**: SonarQube analysis with quality gate check
6. **Docker Build**: Multi-stage Docker image creation
7. **Image Security Scan**: Trivy scan on Docker image (fails on CRITICAL/HIGH)
8. **Push**: Push to DockerHub/Harbor registry
9. **Deploy**: Kubernetes deployment via kubectl

### GitHub Actions Workflow
- **File**: [.github/workflows/github-action-cicd.yml](.github/workflows/github-action-cicd.yml)
- **Trigger**: Push/PR to main branch
- **Runner**: Self-hosted
- **Required Secrets**:
  - `SONAR_TOKEN`, `SONAR_HOST_URL`
  - `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`
  - `KUBE_CONFIG` (base64-encoded kubeconfig)

### Jenkins Pipeline
- **File**: [Jenkinsfile](Jenkinsfile)
- **Registry**: Harbor at `harbor.server.thweb.click`
- **Required Credentials**: `jenkins-harbor-credentials`

### Important Pipeline Notes
- Image tags use Git commit SHA for traceability
- Trivy scans fail pipeline on CRITICAL/HIGH vulnerabilities
- SonarQube quality gate must pass before Docker build
- Deployment uses `boardgame.yaml` Kubernetes manifest
- Pipeline requires self-hosted runners for both GitHub Actions and GitLab

## Development Workflow

### Adding New REST Endpoints
1. Add method to `BoardGameController` or create new `@RestController`
2. Use `DatabaseAccess` for data operations
3. Return `ResponseEntity<?>` with appropriate HTTP status codes
4. Use `ErrorMessage` bean for error responses

### Security Configuration
- Modify `SecurityConfig.configure(HttpSecurity)` to add protected routes
- Role-based access: `hasRole("USER")`, `hasRole("MANAGER")`
- Default users defined in `SecurityConfig.configure(AuthenticationManagerBuilder)`
- CSRF disabled for REST API compatibility

### Database Changes
- Schema changes go in `src/main/resources/schema.sql`
- H2 automatically applies schema on startup (in-memory)

## External Dependencies

### Required Infrastructure
- **SonarQube**: Code quality analysis server (default port 9000)
- **Trivy**: Container and filesystem vulnerability scanner
- **Harbor/DockerHub**: Container registry for image storage
- **Kubernetes Cluster**: KIND or production cluster for deployment
- **Maven Repository**: Nexus at `43.205.242.45:8081` (configured in pom.xml for distribution)

### Maven Distribution
The `pom.xml` includes distribution management for:
- Releases: `http://43.205.242.45:8081/repository/maven-releases/`
- Snapshots: `http://43.205.242.45:8081/repository/maven-snapshots/`

## Implementation Guides

Comprehensive setup guides are available in `docs/`:
- [docs/gitlab-cicd.md](docs/gitlab-cicd.md): Complete GitLab CI/CD setup with runner configuration, SonarQube integration, and Kubernetes deployment
- [docs/github-actions-cicd.md](docs/github-actions-cicd.md): GitHub Actions workflow setup with self-hosted runners and secrets configuration

## Application Deployment Notes

### Kubernetes Deployment Details
- **Namespace**: `boardgame`
- **Replicas**: 2 pods for high availability
- **Resources**: 256Mi-512Mi memory, 100m-500m CPU per pod
- **Health Checks**: Liveness probe (45s initial delay), Readiness probe (30s initial delay)
- **Image Pull**: Requires `harbor-registry` secret for private registry access
- **Ingress**: NGINX ingress with Let's Encrypt TLS certificate

### Environment Variables
- `SPRING_DATASOURCE_URL`: jdbc:h2:mem:testdb
- `SPRING_H2_CONSOLE_ENABLED`: true

### Port Configuration
- Application port: **8080**
- Service port: **80** → **8080**
- Ingress exposes via domain with TLS
