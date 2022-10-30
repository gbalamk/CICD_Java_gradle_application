pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            agent {
                any {
                    image 'openjdk:11'
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
        stage ("Docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-pass', variable: 'docker_pass')]) {
                        sh '''
                    docker build -t 34.125.215.36:8083/springapp:${VERSION} .
                    docker login -u admin -p $docker_pass 34.125.215.36:8083
                    docker push  34.125.215.36:8083/springapp:${VERSION}
                    docker rmi 34.125.215.36:8083/springapp:${VERSION}
                    '''
                    }
                }
            }
        }
        stage("Identifying misconfigs using datree in helm charts"){
            steps{
                script{
                    dir('kubernetes/') {
                        sh 'helm datree test myapp/'
                    }
                }
            }
        }
    
    }
}