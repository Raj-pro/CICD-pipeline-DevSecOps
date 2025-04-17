![Screenshot 2025-04-18 000324](https://github.com/user-attachments/assets/e4c213b3-6140-4ae1-af45-73c03e25a009)# 🚀 DevSecOps CI/CD Pipeline Project

This project demonstrates the implementation of a complete **DevSecOps CI/CD pipeline**, integrating security tools like **SonarQube**, **OWASP Dependency-Check**, and **Trivy** with **Docker** and **Jenkins**. It automates the processes of building, testing, scanning, and deploying applications, ensuring both efficiency and security in the software development lifecycle.

---

## 📺 Project Overview

This project is inspired by the tutorial:
**DevSecOps End to End CICD Project | DevOps Engineer | SonarQube + OWASP + Trivy + Docker + Jenkins**

The goal is to build a secure and automated CI/CD pipeline that ensures code quality, checks for vulnerabilities, and deploys applications seamlessly using the `wanderlust` application as an example.

---

## 🏗️ Technologies Used

-   **Jenkins:** Automation server for CI/CD pipeline orchestration.
-   **Docker:** Containerization platform for building and running applications, including SonarQube.
-   **SonarQube:** Static code analysis tool for code quality and security vulnerabilities.
-   **OWASP Dependency-Check:** Software Composition Analysis (SCA) tool to detect vulnerable dependencies.
-   **Trivy:** Vulnerability scanner for container images and filesystems.
-   **Git:** Version control system for managing source code.
-   **Shell Scripting:** Used for various commands within the pipeline.
-   **Wanderlust App:** The sample application being processed by the pipeline (Node.js based).

---

## ✅ Prerequisites

Before starting, ensure the following are set up:

1.  **Source Code Repository:** Clone the required GitHub repository:
    ```bash
    git clone https://github.com/shubham153/wanderlust
    ```
2.  **Environment:** An AWS EC2 instance, local server, or virtual machine with sufficient resources (e.g., Ubuntu/Debian based).
3.  **Installed Tools:** The following tools need to be installed on the server where Jenkins will run or be accessible to Jenkins agents:
    *   Jenkins
    *   Docker & Docker Compose
    *   SonarQube (can be run via Docker)
    *   OWASP Dependency-Check (Jenkins Plugin)
    *   Trivy

---

## 🔧 Setup Instructions

*(Note: Commands below assume a Debian/Ubuntu-based system. Adjust for your OS if necessary. You might need `sudo` for some commands.)*

### 🛠️ Jenkins Installation & Setup

1.  **Install Java (Required for Jenkins):**
    ```bash
    sudo apt-get update
    sudo apt-get install openjdk-11-jdk -y
    ```
2.  **Install Jenkins:**
    ```bash
    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt-get update
    sudo apt-get install jenkins -y
    ```
3.  **Start Jenkins:**
    ```bash
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
    sudo systemctl status jenkins
    ```
4.  **Access Jenkins:** Open your browser to `http://<your-server-address>:8080`.
5.  **Unlock Jenkins:** Get the initial admin password:
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
    Paste this password into the Jenkins setup screen.
6.  **Install Plugins:** Choose "Install suggested plugins".
7.  **Create Admin User:** Set up your first admin user account.
8.  **Install Necessary Jenkins Plugins:** Go to `Manage Jenkins` > `Manage Plugins` > `Available` tab and install:
    *   `SonarQube Scanner`
    *   `OWASP Dependency-Check Plugin`
    *   *(Pipeline, Git, Docker plugins are usually included in suggested plugins)*

### ⚙️ SonarQube Setup

1.  **Run SonarQube using Docker:**
    ```bash
    # Ensure Docker is installed and running
    sudo systemctl start docker
    sudo systemctl enable docker

    # Run SonarQube container
    docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
    ```
    *(Note: The latest SonarQube version might require specific Java versions or have different setup steps. Check SonarQube documentation if needed.)*
2.  **Access SonarQube:** Open `http://<your-server-address>:9000`. Log in with default credentials: `admin` / `admin`. You'll be prompted to change the password.
3.  **Generate SonarQube Token:**
    *   In SonarQube, go to `Administration` > `Security` > `Users`.
    *   Click the token icon next to the `admin` user (or create a dedicated user).
    *   Generate a token (e.g., name it `jenkins-token`). **Copy this token immediately**, as it won't be shown again.
4.  **Configure SonarQube in Jenkins:**
    *   Go to `Manage Jenkins` > `Configure System`.
    *   Scroll down to the `SonarQube servers` section.
    *   Click `Add SonarQube`.
    *   Name: `SonarQube` (or any name you prefer)
    *   Server URL: `http://<your-server-address>:9000`
    *   Server authentication token: Click `Add` > `Jenkins`.
        *   Kind: `Secret text`
        *   Secret: Paste the SonarQube token you generated.
        *   ID: `sonar-token` (This ID is used in the pipeline script).
        *   Description: `SonarQube Access Token`
        *   Select the newly added credential from the dropdown.
    *   Save the configuration.
5.  **Configure SonarQube Scanner Tool in Jenkins:**
    *   Go to `Manage Jenkins` > `Global Tool Configuration`.
    *   Scroll down to `SonarQube Scanner`.
    *   Click `Add SonarQube Scanner`.
    *   Name: `SonarQube Scanner` (or similar - this name is used in the pipeline `tool` step).
    *   Choose `Install automatically` (select a version) or provide the path if installed manually.
    *   Save the configuration.

### 🛡️ OWASP Dependency-Check Setup

1.  **Plugin Already Installed:** You should have installed the `OWASP Dependency-Check Plugin` earlier.
2.  **Configure Globally (Optional but Recommended):**
    *   Go to `Manage Jenkins` > `Global Tool Configuration`.
    *   Scroll down to `Dependency-Check`.
    *   Click `Add Dependency-Check`.
    *   Name: `Default Dependency Check` (or similar).
    *   Choose `Install automatically` and select a version. This will download the necessary CLI tool.
    *   Configure `Advanced` settings like update frequency if needed.
    *   Save the configuration.
    *(Note: The pipeline step `dependencyCheckPublisher` will use this configuration or trigger scans based on build tool integrations if not configured globally).*

### 🔍 Trivy Setup

1.  **Install Trivy on the Jenkins Server/Agent:**
    ```bash
    # Add Trivy repository and key
    sudo apt-get install wget apt-transport-https gnupg lsb-release -y
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
    echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

    # Update package list and install Trivy
    sudo apt-get update
    sudo apt-get install trivy -y
    ```
2.  **Verify Installation:**
    ```bash
    trivy --version
    ```

### 🐳 Docker & Docker Compose Setup

1.  **Install Docker:** (If not already installed)
    ```bash
    sudo apt-get update
    sudo apt-get install docker.io -y
    sudo systemctl start docker
    sudo systemctl enable docker
    ```
2.  **Add Jenkins User to Docker Group:** (To allow Jenkins to run Docker commands without sudo)
    ```bash
    sudo usermod -aG docker jenkins
    # Restart Jenkins for the group change to take effect
    sudo systemctl restart jenkins
    ```
3.  **Install Docker Compose:**
    ```bash
    # Check the latest release on https://github.com/docker/compose/releases
    LATEST_COMPOSE=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
    sudo curl -L "https://github.com/docker/compose/releases/download/${LATEST_COMPOSE}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    docker-compose --version
    ```

---

## 🧪 Jenkins Pipeline Configuration

1.  **Create a New Jenkins Job:**
    *   In Jenkins, click `New Item`.
    *   Enter a name (e.g., `wanderlust-devsecops-pipeline`).
    *   Select `Pipeline`.
    *   Click `OK`.
2.  **Configure the Pipeline:**
    *   Scroll down to the `Pipeline` section.
    *   Choose `Pipeline script` from the `Definition` dropdown.
    *   Paste the following Groovy script into the `Script` text area.
    *   **Important:** Replace `<your-sonar-server>` with the actual IP address or DNS name of your SonarQube server.

### 📝 Pipeline Script

```groovy
pipeline {
    agent any
    environment {
        SONAR_HOME = tool 'sonar' 
    }

    stages {
        stage('Cone code from github') {
            steps {
                git url: "https://github.com/Raj-pro/CICD-pipeline-DevSecOps.git" , branch: "main"
            }
        }

        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("sonar") {
                    sh "${SONAR_HOME}/bin/sonar-scanner -Dsonar.projectName=Cicdproject -Dsonar.projectKey=Cicdproject"

                }
            }
        }


        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'oqasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan") {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage("Deploy using Docker Compose") {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
}
```
<img src="https://github.com/user-attachments/assets/91713f99-002f-489d-9c74-73079db3879a">
<img src="https://github.com/user-attachments/assets/894259a1-54cf-4fdb-aa04-b96946726f44">
