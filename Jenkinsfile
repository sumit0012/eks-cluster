pipeline{
    
    agent any 
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/sumit0012/demo-counter-app.git'
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
            stage('upload jar file to nexus'){
                
                steps{

                    script{
                        def readPomVersion = readMavenPom file: 'pom.xml'

                        def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "cicd-snapshot" : "cicd"
                        nexusArtifactUploader artifacts: 
                        [
                            [
                                artifactId: 'springboot',
                                classifier: '',
                                file: 'target/Uber.jar',
                                type: 'jar'
                            ]
                        ], 
                            credentialsId: 'nexus-auth', 
                            groupId: 'com.example', 
                            nexusUrl: '34.125.58.111:8081', 
                            nexusVersion: 'nexus3', 
                            protocol: 'http', 
                            repository: 'nexusRepo', 
                            version: "${readPomVersion.version}"
                    }

                }
            }
            stage('Docker Image Build'){
                steps{
                    script {
                        sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker image tag $JOB_NAME:v1.$BUILD_ID sam3ctc/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image tag $JOB_NAME:v1.$BUILD_ID sam3ctc/$JOB_NAME:latest'
                    }
                }
            }
            stage('Push Image to Docker Hub'){
                steps{
                    script{
                        withCredentials([string(credentialsId: 'docker-hub-cred', variable: 'docker-hub-cred')]) {

                            sh 'docker login -u sam3ctc -p ${docker-hub-cred}'
                            sh 'docker image push sam3ctc/$JOB_NAME:v1.$BUILD_ID'
                            sh 'docker image push sam3ctc/$JOB_NAME:latest'
                        }   
                    }
                }
            }
        }
        
}