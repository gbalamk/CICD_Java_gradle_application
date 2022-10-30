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
                    timeout(time: 5, unit: 'MINUTES') {
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
        stage ("Pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-pass', variable: 'docker_pass')]) {
                        dir('kubernetes/') {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d : -f 2 | tr -d ' ')
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u admin:$docker_pass http://34.125.215.36:8081/repository/helm-hosted-bala/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
                    }    
                }   
            }
        }
        // stage("Identifying misconfigs using datree in helm charts"){
        //     steps{
        //         script{
        //             dir('kubernetes/') {
        //                 withEnv(['DATREE_TOKEN=992a00c6-464e-4b9a-93cf-e60009ddb021']) {
        //                     sh 'helm datree test myapp/'
        //                 }
        //             }
        //         }
        //     }
        // }
    
    }
}