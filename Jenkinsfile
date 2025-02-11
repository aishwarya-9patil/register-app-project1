           pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "aishwarya0909"
        DOCKER_PASS = credentials('dockerhub')  // Ensure Docker credentials are properly set in Jenkins
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

        stage("Test JENKINS_API_TOKEN") {
            steps {
                script {
                    // Check if the token is available
                    if (env.JENKINS_API_TOKEN) {
                        echo "JENKINS_API_TOKEN is available."
                    } else {
                        error "JENKINS_API_TOKEN is NOT available."
                    }
                }
            }
        }

        stage("Test Docker Credentials") {
            steps {
                script {
                    echo "Docker Username: ${DOCKER_USER}"
                    // The password won't be exposed, but you can check the username
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Login to Docker registry first
                    docker.login(user: "${DOCKER_USER}", password: "${DOCKER_PASS}")

                    // Build Docker image
                    docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")

                    // Push Docker image to Docker registry
                    docker_image.push("${IMAGE_TAG}")
                    docker_image.push('latest')
                }
            }
        }

        // Add any additional stages here (like Test, Deploy, etc.)
    }
}
