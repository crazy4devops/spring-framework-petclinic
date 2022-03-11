pipeline {
    // agent any
    agent {
        label 'linux-ubuntu'
    }
    environment {
        git_repo_url = "https://github.com/crazy4devops/spring-framework-petclinic.git"
        git_repo_br = "dev"
        tomcat_dev = "ec2-13-233-112-10.ap-south-1.compute.amazonaws.com"
        tomcat_uat = "ec2-13-234-34-51.ap-south-1.compute.amazonaws.com"
        tomcat_prd = "ec2-3-109-139-179.ap-south-1.compute.amazonaws.com"
    }
    triggers {
        cron '* * * * *'
        GenericTrigger(
            genericVariables: [
            [key: 'ref', value: '$.ref']
            ],
            causeString: 'Triggered on $ref',
            token: 'abc123',
            tokenCredentialId: '',
            printContributedVariables: true,
            printPostContent: true,
            silentResponse: false,
            regexpFilterText: '$ref',
         //   regexpFilterExpression: 'refs/heads/' + GIT_BRANCH
        )
    }
    stages{  
        stage("Build Source Code"){
            agent {
                
            }
            environment {

            }
            steps {
                   
                    sh "./mvnw package"
                    sh "mv target/petclinic*.war target/petclinicApp-${BUILD_NUMBER}.war"
            }
        }
        stage('Code Analysis') {
            
            environment {
              def scannerHome = tool 'SonarQubeScanner'
            }
            steps {  
                withSonarQubeEnv('sonarserver') {
                    sh "/opt/sonar/bin/sonar-scanner"
                }
                sleep time: 30000, unit: 'MILLISECONDS'
                script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                }
            }
        }
        stage("Upload Artifacts"){
            steps{
                rtUpload (
                    serverId: 'jfrog-server',
                    spec: '''{
                        "files": [
                            {
                            "pattern": "target/*.war",
                            "target": "dummyrepo/petclinicApp/"
                            }
                        ]
                    }''',
                )  
            }
        }            
        stage("RenameArtifacts"){
            steps {
                sh """
                    mv target/*.war target/petclinicApp.war
                """
            }
        }
        stage("deploy-aws-dev"){
            steps{
                sshagent(['aws-ec2-creds']) {
                    sh """
                        scp  -r -o StrictHostKeyChecking=no target/*.war   ubuntu@\${tomcat_dev}:/opt/tomcat/webapps
                    """
                }
            }
        }
        stage("deploy-aws-uat"){

            steps{
                sshagent(['aws-ec2-creds']) {
                        sh """ 
                           scp  -r -o StrictHostKeyChecking=no target/*.war   ubuntu@\${tomcat_uat}:/opt/tomcat/webapps
                        """
                }
            }
        }
        stage("deploy-aws-prd"){
            input{
                    message "Do you want to proceed for production deployment?"
            }
            steps{
                sshagent(['aws-ec2-creds']) {
                        sh """
                            scp  -r -o StrictHostKeyChecking=no target/*.war   ubuntu@\${tomcat_prd}:/opt/tomcat/webapps
                        """
                }
            }
        }
    }
        always { 
            cleanWs()
        }
    }
}
//ssh -tt -o StrictHostKeyChecking=no ubuntu@\${tomcat_prd}; sudo systemctl stop tomcat; sudo rm -rf /opt/tomcat/webapps/petclinic*;exit
// stage("Run Unit-Tests"){
        //     steps {
        //             sh "./mvnw test"
        //     }
        // }
//  ssh ubuntu@ec2-65-0-95-227.ap-south-1.compute.amazonaws.com; sudo systemctl restart tomcat
// stage("Checkout"){
        //     steps {
        //         // Clean before build
        //         cleanWs()
        //         git url: "${git_repo_url}", branch: "${git_repo_br}"
        //         echo 'Pulling...' + env.BRANCH_NAME
        //     }
        // }
 // stage("Deploy - DEV"){
        //     steps{
        //         echo "Running Deployment on Dev"
        //         sh """
        //           sshpass -p 'Venkat@123' scp -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no target/*.war cloud_user@172.31.46.115:/opt/tomcat/webapps
        //         """
        //     }
        // }
