pipeline {
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "aishwarya0909"  // Docker username
        DOCKER_PASS = credentials('Jenkins-docker-password')  // Docker password from Jenkins credentials
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/aishwarya-9patil/register-app-project1.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Debugging Step
                    echo "DOCKER_USER: ${DOCKER_USER}"

                    // Log in to Docker Hub using the Docker username and password from Jenkins
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_PASS) {
                        // Build Docker image
                        docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    }

                    // Push Docker image to Docker registry (Docker Hub)
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_PASS) {
                        // Push the image with both the specific tag and 'latest'
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        // Additional stages can go here (like Test, Deploy, etc.)
    }
}
           

         
