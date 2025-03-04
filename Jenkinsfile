pipeline {
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
	stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/vishwanath0303/Petclinic.git'

            }
        }
   
         stage('Compile') {
            steps {
               sh "mvn compile"
            }
        }
         stage('Test') {
            steps {
                sh "mvn clean install"
                }
        }
         stage('Build & tag docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t vkulkarni0303/petclinic:latest ."
    
            
                        
                    }
            }
            }
        }
        stage('Push docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push vkulkarni0303/petclinic:latest "
                        
                    }
            }
            }
        }

	    stage('Clean up containers') {   //if container runs it will stop and remove this block
          steps {
           script {
             try {
                sh 'docker stop petclinic '
                sh 'docker rm petclinic '
                } catch (Exception e) {
                  echo "Container pet1 not found, moving to next stage"  
                }
            }
          }
        }
		stage("Deploy "){
            steps{
                sh "docker run --name petclinic -d -p 8082:8080  vkulkarni0303/petclinic:latest "
            }
        }
	}
}
