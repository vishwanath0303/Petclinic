pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
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
		 stage('Publish to nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true)  {
                 sh "mvn deploy -X"

             }
            }
        }	
		stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('sonar') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
         stage('Build & tag docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t vkulkarni0303/petclinic:$BUILD_NUMBER ."
    
            
                        
                    }
            }
            }
        }
        stage('Push docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push vkulkarni0303/petclinic:latest"
                        
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
                sh "docker run --name petclinic -d -p 8082:8080  vkulkarni0303/petclinic:$BUILD_NUMBER "
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.57.191:6443') {
    			sh " kubectl apply -f deployment-service.yaml "
 
}
            }
	}
		stage('Verify the aDeployment') {
            steps { 
		    withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.57.191:6443') {
				    sh "kubectl get pods"
				    sh "kubectl get svc -n webapps"
}
            }
		}
         
    }
}
