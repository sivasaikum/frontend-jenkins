pipeline {
    agent { label 'STAGE-1'}
    environment {
        PROJECT = 'expense'
        COMPONENT = 'frontend'
        appVersion = ''
        ACC_ID = '381491879282'
    }
    options {
        timeout(time:30 , unit:'MINUTES')
    }
    parameters {
        booleanParam(name:'deploy', defaultValue: 'false', description: 'select when you are ready to deploy')
    }

    stages {
        stage('Read Json Version') {
            steps {
                script {
                    def JsonRead = readJSON file: 'package.json'
                    appVersion = JsonRead.version
                    echo " Version is $appVersion"
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    withAWS(region: 'us-east-1',  credentials: 'aws-cred') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                            echo "docker image is successfuly puhed to AWS ECR"
                        """
                    }
                }
            }
        }

        stage('Trigger Deploy') {
            when {
                expression {
                    params.deploy
                }
            }
            steps {
                script {
                    build job: 'frontend-ci', parameters: [string(name: 'version', value: "${appVersion}")], propagate: true
                }
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure { 
            echo 'I will run when pipeline is failed'
        }
        success { 
            echo 'I will run when pipeline is success'
        }
    }
}