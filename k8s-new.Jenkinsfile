pipeline {
    agent any
    environment {
            registry_url = "http://184.72.168.66:80"
            registry = "184.72.168.66:80/javaapp/petapp"
            registryCredential = 'venkat-harborhub'
            dockerImage = ''
    }
    stages{
        stage("buildApp"){
            steps{
                script {
                    sh """./mvnw package
                         mv ./target/*.war ./target/petApp-${BUILD_NUMBER}.war
                         ls -lrt ./target/petApp*.war
                    """
                    
                }
            }
        }
        stage('Building image') {
            steps{
              script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
              }
            }
        }
        stage('Deploy Image') {
          steps{
            script {
              docker.withRegistry(registry_url, registryCredential ) {
                dockerImage.push()
              }
            }
          }
        }
        stage('Remove Old Images') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
        stage("update-deployment"){
            steps {
                sshagent(credentials: ['ec2-user']) {
                    script {
                        sh """                    
                             sed -i 's/petapp:latest/petapp:${env.BUILD_NUMBER}/g' k8s-deployments/petclinicapp-deploy.yaml
                             scp -o StrictHostKeyChecking=no k8s-deployments/petclinicapp-deploy.yaml ubuntu@ec2-54-235-37-175.compute-1.amazonaws.com:/home/ubuntu
                        """
                    }
                       
                }
            }
        }
        stage(" execute Ansible") {
          steps {
                ansiblePlaybook credentialsId: 'ec2-user', disableHostKeyChecking: true, installation: 'Ansible', inventory: 'ansible-deploy/sinventory', playbook: 'ansible-deploy/k8s-deploy.yaml'
          }    
        }  
        
    }

    post {
        always {
           cleanWs()
        }
    }
}
