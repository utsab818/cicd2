pipeline {
    agent any
    environment {
        VERSION = "$(env.BUILD_ID)"
    }

    stages{
        stage('Sonar quality check'){
            agent{
                docker{
                    image 'maven'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token'){
                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
        stage('Quality gate status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage('docker build and push to nexus repo'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]){
                        sh '''
                        docker build -t #nexus_ip/sprinapp:$(VERSION) .
                        docker login -u admin -p $nexus_creds #nexus_ip
                        docker push #nexus_ip/sprinapp:$(VERSION)
                        docker rmi #nexus_ip/sprinapp:$(VERSION
                        '''
                    }
                }
            }
        }
    }
}