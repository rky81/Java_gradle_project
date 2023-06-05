pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage ("Sonarqube quality check"){
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
        stage("docker build and docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass')]) {
                        sh '''
                            docker build -t 34.125.25.198:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus_pass 34.125.25.198:8083
                            docker push 34.125.25.198:8083/springapp:${VERSION}
                            docker rmi 34.125.25.198:8083/springapp:${VERSION}
                            '''

                    }
                    
                }
            }
        }
    }
}