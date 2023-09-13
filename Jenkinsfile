pipeline {
    agent any
     tools{
        jdk  'jdk17'
        maven  'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/ashush77/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
               sh "mvn compile"
            }
        }
        
         stage('Trivy FS Scan') {
            steps {
               sh "trivy fs ."
            }
        }
        
         stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
               }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
         stage('Deploy to Nexus') {
            steps {
              withMaven(globalMavenSettingsConfig: 'global-config') {
               sh "mvn deploy -DskipTests=true"
              }  
            }
        }
        
         stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart ashushrm77/shopping-cart:latest"
                       
                    }
                }
            }
        }
        
         stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        
                        sh "docker run -d --name ekart -p 8070:8070 ashushrm77/shopping-cart:latest"
                    }
                }
            }
        }
        
         stage('Deploy Application') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        
                        sh "docker push ashushrm77/shopping-cart:latest"
                    }
                }
            }
        }
        
        
    }
}

        
        
    }
}
