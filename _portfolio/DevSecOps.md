---
header:
  overlay_image: assets/images/DevSecOps/project_overview.png
  caption: "Photo credit: [**Marcus Spiske**](https://unsplash.com)"
permalink: /portfolio/DevSecOps/
date: 2024-11-10
toc: true
toc_label: "Contents"
---
# CI/CD Pipeline for "tp foyer" Application 🌐🚀

## Overview
Welcome! This project showcases a CI/CD pipeline I created for the "tp foyer" application. It was developed as an academic project, combining my interests in DevOps and cybersecurity with a focus on secure, efficient, and monitored deployment processes.

   - ![project overview](/assets/images/DevSecOps/project_overview.png)

**Note**: If you're interested in a full exported version of this project, please feel free to contact me!

## Project Goals
The primary goals for this project were to:
- **Automate** the entire development, testing, and deployment cycle.
- **Secure** the pipeline to prevent vulnerabilities at every stage.
- **Monitor** the application in real time for efficient incident response.

## Architecture & Tools 🛠️
For this pipeline, I used a selection of DevOps tools to achieve reliable and secure CI/CD:

- **Git**: Version control to keep track of code changes.
- **Maven**: Manages compilation and dependencies.
- **JUnit/Mockito**: Unit testing for code reliability and accuracy.
- **SonarQube**: Analyzes code quality and security vulnerabilities.
- **Nexus**: Artifact repository for hosting project artifacts.
- **Docker & DockerHub**: Containerization for consistent deployment.
- **Docker Compose**: Manages container orchestration and dependencies.
- **Prometheus & Grafana**: Real-time performance monitoring and visualization.

## Pipeline Stages 🛠️

### 1. **Source Code Management with Git**
   - **Git** tracks code changes and facilitates collaboration. Each commit automatically triggers the CI/CD pipeline to build, test, and deploy the latest code.

### 2. **Build with Maven**
   - **Maven** automates the build process, handling code compilation and dependency management. This step ensures that the codebase is consistent, and dependencies are up to date.

### 3. **Testing with JUnit and Mockito**
   - **JUnit and Mockito** run unit tests to validate code functionality, catching bugs early in development.

### 4. **Code Quality & Security with SonarQube**
   - **SonarQube** performs static code analysis to detect code quality issues and security vulnerabilities, aligning the project with secure coding best practices.
   - ![SonarQube Analysis](/assets/images/DevSecOps/sonar.png)

### 5. **Artifact Management with Nexus**
   - After a successful build and testing phase, artifacts (e.g., JAR files) are pushed to **Nexus**. Nexus serves as a repository, allowing seamless artifact management and access in later stages.
   - ![Nexus Repository](/assets/images/DevSecOps/nexus.png)

### 6. **Containerization with Docker & DockerHub**
   - The application is **containerized** with Docker, creating a standardized environment for deployment.
   - Containers are stored in **DockerHub**, making them accessible for deployment in various environments.

### 7. **Container Orchestration with Docker Compose**
   - **Docker Compose** manages multiple containers, handling dependencies between different services and ensuring the app runs consistently across environments.

### 8. **Real-Time Monitoring with Prometheus & Grafana**
   - **Prometheus** monitors the application’s performance and health metrics.
   - ![Prometheus Monitoring](/assets/images/DevSecOps/Prometheus.png)

   - **Grafana** provides a visual dashboard to track CPU usage, memory, response time, and more, enabling proactive monitoring and quick response to issues.

   - ![Grafana Jenkins Dashboard](/assets/images/DevSecOps/grafana_jenkins.png)


   - ![Grafana Database Size](/assets/images/DevSecOps/grafana_db_size.png)

## Pipeline Overview
The CI/CD pipeline integrates each tool and stage above to ensure a secure, automated, and monitored deployment of the "tp foyer" application.
   - ![Pipeline Overview](/assets/images/DevSecOps/pipline_overview.png)

#### Docker Compose YAML
```yaml
version: "3.8"

services:
  mysqldb:
    image: mysql:5.7
    restart: unless-stopped
    environment:
      - MYSQL_ALLOW_EMPTY_PASSWORD=1
      - MYSQL_DATABASE=tp foyer_db
    ports:
      - "3306:3306"
    volumes:
      - db:/var/lib/mysql

  tp-foyer-app:
    depends_on:
      - mysqldb
    # build:
    #   context: .
    image: abbesachraf/tp-foyer:5.0.0
    restart: on-failure
    ports:
      - "8089:8089"
    environment:
      SPRING_APPLICATION_JSON: '{
          "spring.datasource.url": "jdbc:mysql://mysqldb:3306/tp foyer_db?createDatabaseIfNotExist=true",
          "spring.datasource.username": "root",
          "spring.datasource.password": null,
          "spring.jpa.show-sql": true,
          "spring.jpa.hibernate.ddl-auto": "update"
        }'
    stdin_open: true
    tty: true

volumes:
  db:
```

#### Jenkins final Pipline
```groovy
pipeline {
    agent any
        stage('Setup Environment - Start Containers') {
            steps {
                script {
                    def prometheusStatus = sh(script: 'docker inspect -f "{{.State.Running}}" 4fb9f43af657 || echo "false"', returnStdout: true).trim()
                    if (prometheusStatus != "true") { sh 'docker start 4fb9f43af657'; echo 'Prometheus started.' }
                    
                    def grafanaStatus = sh(script: 'docker inspect -f "{{.State.Running}}" ae9711f381a5 || echo "false"', returnStdout: true).trim()
                    if (grafanaStatus != "true") { sh 'docker start ae9711f381a5'; echo 'Grafana started.' }
                    
                    def additionalContainerStatus = sh(script: 'docker inspect -f "{{.State.Running}}" bf03926b0c4b || echo "false"', returnStdout: true).trim()
                    if (additionalContainerStatus != "true") { sh 'docker start bf03926b0c4b'; echo 'Additional container started.' }

                    def nexusStatus = sh(script: 'docker inspect -f "{{.State.Running}}" dfbc396469ec || echo "false"', returnStdout: true).trim()
                    if (nexusStatus != "true") { sh 'docker start dfbc396469ec'; echo 'Nexus container started.' }

                    def fail2banStatus = sh(script: 'systemctl is-active --quiet fail2ban && echo "running" || echo "not running"', returnStdout: true).trim()
                    echo "Fail2Ban status: ${fail2banStatus}"
                }
                echo 'Finished setting up environment...'
            }
        }
        stage('teeeeeeeeeeeeeeeeeeest Git Checkout') {
            steps {
                git branch: 'Achraf', url: 'https://github.com/abbesachraf/tp-foyer.git'
                echo 'fiiiiinish Pulling the repository...'
            }
        }
        stage('teeeeeeeeeeeeeeeeeeest Maven Clean, Compile, and Package') {
            steps {
                sh 'mvn clean compile'
                sh 'mvn clean package'
                echo 'fiiiiinish Running Maven clean, compile, and package...'
            }
        }
        stage('teeeeeeeeeeeeeeeeeeest Sonar Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SonarQube_jenkins', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=tp-foyer -Dsonar.host.url=http://192.168.50.4:9000 -Dsonar.login=$SONAR_TOKEN'
                }
                echo 'fiiiiinish Running SonarQube analysis...'
            }
        }
        stage('teeeeeeeeeeeeeeeeeeest Docker Build and Push') {
            steps {
                sh 'sudo docker build -t abbesachraf/tp-foyer:5.0.0 .'
                echo 'fiiiiinish Running Docker image build...'
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh 'docker push abbesachraf/tp-foyer:5.0.0'
                    echo 'fiiiiinish Running Docker Push...'
                }
            }
        }
        stage('teeeeeeeeeeeeeeeeeeest Nexus Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id', passwordVariable: 'NEXUS_PASS', usernameVariable: 'NEXUS_USER')]) {
                    sh 'mvn deploy -DskipTests -Dusername=$NEXUS_USER -Dpassword=$NEXUS_PASS'
                    echo 'fiiiiinish Running Nexus deploy...'
                }
            }
        }
    }
}
```

## Results & Key Takeaways
This project not only automated the deployment process but also built security and quality checks into each stage. Here are some of the key takeaways:

- **Automation Increases Reliability**: Automating the pipeline reduced human error, ensuring consistent, reliable deployments.
- **Security Built-In**: SonarQube integration helps catch security vulnerabilities early, enhancing the application’s resilience.
- **Proactive Monitoring**: Prometheus and Grafana enabled real-time monitoring, allowing for quick responses to potential issues.

## Conclusion
Creating this CI/CD pipeline for "tp foyer" was a rewarding experience that enhanced my DevOps knowledge and gave me practical experience with industry-standard tools. I’m excited to keep exploring DevOps and cybersecurity intersections, applying these practices to deliver secure and efficient software.

Thank you for reading! Feel free to reach out if you have questions about any part of the pipeline or would like a complete exported version of this academic project.

---

**Hashtags**: #DevOps #Cybersecurity #CICD #Automation #Monitoring
