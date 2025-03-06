pipeline {
    agent any
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/vishwanath0303/Petclinic.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    // Use withCredentials to securely inject Docker Hub credentials
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', 
                                                       usernameVariable: 'DOCKER_USER', 
                                                       passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Log into Docker Hub
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin'
                        // Build the Docker image
                        sh 'docker build -t vkulkarni0303/petclinic:latest .'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', 
                                                       usernameVariable: 'DOCKER_USER', 
                                                       passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Log into Docker Hub (again, to ensure authentication)
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin'
                        // Push the Docker image
                        sh 'docker push vkulkarni0303/petclinic:latest'
                    }
                }
            }
        }
        stage('Clean Up Containers') {
            steps {
                script {
                    try {
                        sh 'docker stop petclinic'
                        sh 'docker rm petclinic'
                    } catch (Exception e) {
                        echo "Container 'petclinic' not found, moving on..."
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker run --name petclinic -d -p 8082:8080 vkulkarni0303/petclinic:latest'
            }
        }
    }
}
