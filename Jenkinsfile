pipeline {
    agent { label 'AGENT-1'}
    environment { 
        PROJECT = 'expense'
        COMPONENT = 'backend' 
        DEPLOY_TO = "production"
        appVersion = ''
        environment = ''
        REGION='us-east-1'

    }

     options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters{
        string(name: 'version',  description: 'Enter the application version')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick something')
    }
    stages {
        stage('Setup Environment'){
            steps{
                script{
                    appVersion = params.version
                    environment = params.deploy_to
                }
            }
        }

        stage('Deploy') {
            steps {
                script{
                    withAWS(region: 'us-east-1', credentials: "aws-creds-${environment}") {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name expense-${environment}
                            kubectl get nodes
                            cd helm
                            sed -i 's/IMAGE_VERSION/${params.version}/g' values-${environment}.yaml
                            helm upgrade --install $COMPONENT -n $PROJECT -f values-${environment}.yaml .
                        """
                    }
                }
            }
        }

        stage('Functional/API Tests') {
            when{
                expression {params.deploy_to == 'dev'}
            }
            steps {
                script{
                    sh """
                       echo "functional test will be performed after DEV deployment. usually these are automated 
                       selenium test cases written by testing team. If these test cases are failed Pipeline also fails"
                    """
                }
            }
        }
    }
        
    post { 
    
        always { 
            echo 'I will always say hello again!'
            deleteDir()
        }
        failure { 
            echo 'This session runs when pipeline failure'
        }
        success { 
            echo 'This session runs when pipeline success'
        }
    }
}