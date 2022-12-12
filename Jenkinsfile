pipeline{
    
    agent any 
    
    stages {
        
        stage('Git Checkout')
        {
            
            steps{
                
                    script{
                    
                            git branch: 'main', url: 'https://github.com/vijay24794/demo-counter-app.git'
                    }
            }
        }
        stage('UNIT testing')
        {
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing')
        {
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Maven build')
        {
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static code analysis')
        {
            
            steps{
                
                script{
                    
                        withSonarQubeEnv(credentialsId: 'sonar-api')
                        {
                        
                        sh 'mvn clean package sonar:sonar'
                        }
                }
            }
        }
        stage('Quality Gate Status')
        {
                
            steps{
                    
                script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                }
            }
        }
        stage('upload war file to nexus')
        {

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
        stage('Docker Image Build')
        {
            steps{
                sh 'docker info'
                sh 'docker image build -t demoapp:v1.$BUILD_ID .'
                sh 'docker tag demoapp:v1.$BUILD_ID vijay24794/demoapp:v1.$BUILD_ID'
                sh 'docker tag demoapp:v1.$BUILD_ID vijay24794/demoapp:latest'
            }
        }
      
      stage('Docker Image Push to docker Hub ')
      {
            steps{
                
                  script{
                    
                            withCredentials([string(credentialsId: 'docker_password', variable: 'docker_pass')]) 
                            {
                               sh 'docker login -u vijay24794 -p ${docker_pass}'
                               sh 'docker push vijay24794/demoapp:latest'
                            }
                        }
                   }
      }
      stage('Identifying misconfigs using datree in helm charts'){
        steps{
            script{
                dir('kubernets/myapp/'){
                    withEnv(['DATREE_TOKEN=9dfd7b8c-897f-494f-a4a4-57aea61f4bc2']) {
                    sh 'helm datree test .'
                    }
                }
            }

        }
      }
      stage('Pushing the helm charts to nexus repo'){
      steps{
        script{
            withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_creds')]){
                dir('kubernets/myapp/')
                    sh '''
                    helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                    tar -czvf myapp-${helmversion}.tgz myapp/
                    curl -u admin:$nexus_creds http://nexus.devops.com/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                    '''
            }
        }
      }
     }
    }
    post {
	        always {
		                mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "vijay24794@gmail.com";  
		            }
	    }
}
