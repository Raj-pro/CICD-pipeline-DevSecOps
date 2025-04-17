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
