pipeline {    
    agent any 
    tools {
        jdk 'JDK-17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {   
        stage('git-checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/sivaganesh2312/broadGame.git'
            }
        }

        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

  

        stage('sonarqube-analisys') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar-token') { // You can override the credential to be used
                sh 'mvn package -DskipTests=true sonar:sonar -Dsonar.host.url=https://sonarcloud.io -Dsonar.organization=aug2024 -Dsonar.projectKey=BoardGame'}
            }
        }


        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t ganesh231/boardgame:latest ."
                    }
               }
            }
        }

        stage('File System Scan') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html ganesh231/boardgame:latest"
            }
        }
        
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push ganesh231/boardgame:latest"
                    }
               }
            }
        }
        stage('Deploy to kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://10.138.0.4:6443') {
                        sh "kubectl apply -f deployment-service.yaml "
               } 
            }
        }
        stage('Verify the Deployment') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://10.138.0.4:6443') {
                        sh "kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
                }
            }
        } 
    }
}
