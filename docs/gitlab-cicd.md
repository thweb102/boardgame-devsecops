# 🚀 GitLab CI/CD Pipeline Implementation

> A comprehensive guide to implementing a complete DevSecOps CI/CD pipeline using GitLab with Maven, Trivy, SonarQube, Docker, and Kubernetes.

## 📋 Prerequisites

- GitLab account
- Linux VM for GitLab Runner
- Docker installed
- kubectl and kind installed
- Basic knowledge of CI/CD concepts

## 🔧 Implementation Steps

### 1️⃣ Project Import

Import your project repository into GitLab:

```bash
https://github.com/atkaridarshan04/GitLab-CICD.git
```

### 2️⃣ GitLab Runner Configuration

#### 🖥️ Prepare the VM Environment

```bash
sudo apt update && sudo apt upgrade -y
```

#### 🏃‍♂️ Register GitLab Runner

1. Navigate to **Settings** → **CI/CD** → **Runners** in your GitLab project
   
   ![Navigate to Runners](./images/runner_1.png)

2. Follow the runner registration process:
   
   ![Provide Runner Tag](./images/runner_2.png)
   ![Runner Installation Step 1](./images/runner_3.png)
   ![Runner Installation Step 2](./images/runner_4.png)
   ![Runner Installation Step 3](./images/runner_5.png)

---

### 3️⃣ SonarQube Setup & Integration

#### 🐳 Deploy SonarQube with Docker

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```

> 🔐 **Default Credentials:** Username: `admin` | Password: `admin`

![SonarQube Deployment](./images/sonar_1.png)

#### 🔑 Configure SonarQube Integration

1. **Generate Personal Access Token (PAT):**
   
   ![Create PAT Step 1](./images/sonar_2.png)
   ![Create PAT Step 2](./images/sonar_3.png)
   ![Create PAT Step 3](./images/sonar_4.png)
   ![Create PAT Step 4](./images/sonar_5.png)
   ![Create PAT Step 5](./images/sonar_6.png)
   ![Create PAT Step 6](./images/sonar_7.png)

2. **Create `sonar-project.properties` file:**
   
   ![Edit sonar-project.properties](./images/sonar_08.png)
   ![sonar-project.properties Example](./images/sonar_8.png)

3. **Add PAT to GitLab CI/CD Variables:**
   
   ![Add PAT Step 1](./images/sonar_09.png)
   ![Add PAT Step 2](./images/sonar_9.png)
   ![Add PAT Step 3](./images/sonar_10.png)
   ![Add PAT Step 4](./images/sonar_11.png)
   ![Add PAT Step 5](./images/sonar_12.png)
   ![Add PAT Step 6](./images/sonar_13.png)

4. **Configure Pipeline Stage:**
   
   ![Pipeline Stage Example](./images/sonar_14.png)
   
   > ⚠️ **Important:** Remove the entrypoint for the SonarQube container
   
   ![Entrypoint Removal Example](./images/sonar_15.png)

---

### 4️⃣ Kubernetes Cluster Configuration

#### 🎯 Create KIND Cluster

```bash
kind create cluster --config kind-config.yml
```

**kind-config.yml:**
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
    - containerPort: 30080  
      hostPort: 30080
      protocol: TCP
```

#### 🔧 Configure Kubernetes Access

1. **Get kubeconfig:**
   ```bash
   cd $HOME/.kube
   ```

2. **Encode configuration:**
   ```bash
   echo -n "$(cat config)" | base64
   ```

3. **Add to GitLab Variables:**
   
   Create a CI/CD variable named `KUBECONFIG_CONTENT` with the encoded config:
   
   ![Encoded Variable paste](./images/k8s_1.png)
   ![All variables](./images/all_variables.png)

---

### 5️⃣ Pipeline Execution

#### 🚀 Run the Pipeline

1. Execute the pipeline using [.gitlab-ci.yml](.gitlab-ci.yml)
2. Monitor pipeline stages for successful completion

![Pipeline Status](./images/pipeline_status.png)



## 🎯 Pipeline Stages Overview

The GitLab CI/CD pipeline includes the following stages:

- **🔨 Build** - Maven compilation and packaging
- **🔍 Test** - Unit test execution
- **🛡️ Security Scan** - Trivy vulnerability scanning
- **📊 Code Quality** - SonarQube analysis
- **🐳 Docker Build** - Container image creation
- **🚀 Deploy** - Kubernetes deployment

