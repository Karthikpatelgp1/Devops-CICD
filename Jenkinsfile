pipeline {
    agent any
    
    tools {
        jdk 'JDK11'
        maven 'Maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('git-checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Karthikpatelgp1/Devops-CICD.git'
            }
        }
        
        stage('Code-Compile') {
            steps {
               sh "mvn clean compile"
            }
        }
        
        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Devops-CICD \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Devops-CICD '''
                }
            }
        }
        
        stage('TRIVY SCAN') {
            steps {
                sh ' trivy fs --security-checks vuln,config /var/lib/jenkins/workspace/CICD '
            }
        }
        
        stage('CODE BUILD') {
            steps {
                sh " mvn clean install "
            }
        }
        
        stage('DOCKER BUILD') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '9dca081d-6bd7-4f38-8186-a8eb7f3317ac', toolName: 'docker-latest') {
                        sh "docker build -t cicddevops . "
                    }
                    
                }
            }
        }
        
        stage('DOCKER PUSH') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '9dca081d-6bd7-4f38-8186-a8eb7f3317ac', toolName: 'docker-latest') {
                        sh "docker tag cicddevops karthikpatelgp1/cicddevops:$BUILD_ID"
                        sh "docker push karthikpatelgp1/cicddevops:$BUILD_ID"
                    }
                    
                }
            }
        }
        
        stage('DEPLOY TO K8S') {
            steps {
                script{
                    kubernetesDeploy(configs: 'deploymentservice.yaml', kubeconfigId: 'kubernetes')
                }
            }
        }
    }
}
    
  
