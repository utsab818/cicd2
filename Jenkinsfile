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
        stage('Identify misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/myapp/'){
                        withEnv(['DATREE_TOKEN=###YOUR_TOKEN'])
                        sh 'helm datree test .'
                    }
                }
            }
        }

        stage('Pushing the helm charts to nexus repo'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]){
                        dir('kubernetes/'){
                     sh '''
                        helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d '')
                        tar -czvf myapp-${helmversion}.tgz myapp/
                        curl -u admin:$nexus_password http://nexus_machine_ip:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                        '''
                        }
                    }
                }
            }
        }

        post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "utsabsapkota4231@gmail.com";  
		}
	}
    }
}