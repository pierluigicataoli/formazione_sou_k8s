pipeline {
    agent {
        label 'jenkins-agent-02'
    }

    env {
        DOCKER_HUB_USER = 'pcata'
        IMAGE_NAME = "flask-app-example"
        BUILD_TARGET = 'builder'
        DOCKERHUB_CRED_ID = 'dockerhub-pcata-id' 
    }

    stages { 

        stage('Checkout Code') {
            steps {
                echo "Clono il codice su agent ..."
                checkout scm 
            }
        }
    
        stage('Set Image Tag') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.IMAGE_TAG = 'latest'
                    } else {
                        env.IMAGE_TAG = "dev-${env.BUILD_NUMBER}"
                    }
                    echo "Docker image tag set to: ${env.IMAGE_TAG}"
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Buildo l'immagine ${env.DOCKER_HUB_USER}/${env.IMAGE_NAME}:${env.IMAGE_TAG}..."
                bash "docker build --target ${env.BUILD_TARGET} -t ${env.DOCKER_HUB_USER}/${env.IMAGE_NAME}:${env.IMAGE_TAG} ."
            }
        }
    
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.DOCKERHUB_CRED_ID,
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        echo "Logging into Docker Hub and pushing image..."
                        
                        bash "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        
                        bash "docker push ${env.DOCKER_HUB_USER}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                        
                        bash "docker logout"
                    }
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up local image from agent...'
                bash "docker rmi ${env.DOCKER_HUB_USER}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
            }
        }
    } 
} 
