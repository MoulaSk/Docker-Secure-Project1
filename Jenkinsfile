pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        SONARQUBE_TOKEN = credentials('sonarqube-token')
        APP_NAME = "nodejs-app"
        DOCKER_IMAGE = "moulask786k/${APP_NAME}:${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'Social-Prachar', 
                url: 'https://github.com/MoulaSk/Docker-Secure-Project1.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'reports/**/*.xml'
                }
            }
        }
        
        stage('Code Quality - SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh 'sonar-scanner \
                        -Dsonar.projectKey=nodejs-app \
                        -Dsonar.projectName=NodeJS-App \
                        -Dsonar.projectVersion=${BUILD_NUMBER} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONARQUBE_SERVER} \
                        -Dsonar.login=${SONARQUBE_TOKEN} \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info'
                }
            }
        }
        
        stage('Security Scan - Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format HTML --format JSON --out ./reports/dependency-check', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: 'reports/dependency-check/dependency-check-report.json'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }
        
        stage('Container Security Scan - Trivy') {
            steps {
                sh "trivy image --security-checks vuln --exit-code 1 --severity CRITICAL ${DOCKER_IMAGE}"
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Production-like Environment') {
            steps {
                sh """
                    docker stop ${APP_NAME} || true
                    docker rm ${APP_NAME} || true
                    docker run -d --name ${APP_NAME} -p 3000:3000 ${DOCKER_IMAGE}
                """
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(color: 'good', message: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            slackSend(color: 'danger', message: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}
