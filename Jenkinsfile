pipeline {
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "aishwarya0909"  // Docker username
        DOCKER_PASS = credentials('dockerhub1')  // Use the correct credential ID for Docker Hub credentials
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        // JENKINS_API_TOKEN is assumed to be used elsewhere
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()  // Cleanup workspace to ensure a fresh build
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/aishwarya-9patil/register-app-project1.git'
            }
        }

        stage('Build Application') {
            steps {
                sh "mvn clean package"  // This should use the Maven tool configured in Jenkins
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'  // Wait for the quality gate to pass
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    echo "Using Docker registry: https://index.docker.io/v1/"
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"

                    // Ensure Docker login is successful (echo username for debugging)
                    echo "DOCKER_USER: ${DOCKER_USER}"

                    // Log in to Docker Hub using Docker credentials stored in Jenkins
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub1') {
                        // Build Docker image with the specific tag
                        docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    }

                    // Push the Docker image to Docker Hub registry
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub1') {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')  // Optionally, also push the 'latest' tag
                    }
                }
            }
        }

        // Additional stages can go here (e.g., Test, Deploy, etc.)
    }
}
