pipeline{
    
    agent any 
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/vijay24794/demo-counter-app.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static code analysis'){
            
            steps{
                
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonar-api') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                }
            }
            stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                    }
                }
            }
        stage('upload war file to nexus'){

            steps{

                script{
                    
                    
                    nexusArtifactUploader artifacts: 
                     [
                         [
                             artifactId: 'springboot', 
                             classifier: '', file: 'target/Uber.jar', 
                             type: 'jar'
                         ]
                     ], 
                        credentialsId: 'nexus-auth', 
                        groupId: 'com.example', 
                        nexusUrl: '172.31.46.147:8081', 
                        nexusVersion: 'nexus3', 
                        protocol: 'http', 
                        repository: 'demoapp-release', 
                        version: "v1.$BUILD_ID"
                }
            }
        }
        stage('Docker Image Build'){
            steps{
                sh 'docker info'
                sh 'docker image build -t demoapp:v1.$BUILD_ID .'
                sh 'docker tag demoapp:v1.$BUILD_ID vijay24794/demoapp:v1.$BUILD_ID'
                sh 'docker tag demoapp:v1.$BUILD_ID vijay24794/demoapp:latest'
            }
          }
      
      stage('Docker Image Push to docker Hub '){

            steps{
                        sh 'docker login -u vijay24794@gmail.com -p Vijay@1994'
                        sh 'docker push vijay24794/demoapp:latest'
                      
                       }
                }
    }
}
