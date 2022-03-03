pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
        url = "${env.BUILD_URL}"
    }

    stages{
        stage("Sonar Quality Check"){
            agent{
                docker {
                    image 'openjdk:8'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }

                }
            }

        }

        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                             sh '''
                                docker build -t 18.169.53.99:8083/springapp:${VERSION} .
                                docker login -u admin -p $docker_password 18.169.53.99:8083 
                                docker push  18.169.53.99:8083/springapp:${VERSION}
                                docker rmi 18.169.53.99:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{

                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=fsQNDSGM8uAhuLwXfsuSWU']) {
                            sh 'helm datree test /var/lib/jenkins/workspace/k8s_test_cicd/kubernetes/myapp'
                        }
                    }
                }
            }
        }

        
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${url}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "gowthampreparation@gmail.com";  
		 }
	   }    
}