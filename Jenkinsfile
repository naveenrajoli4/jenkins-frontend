pipeline {
    agent {
        label 'agent-1-label'
    }
    options{
        timeout(time: 50, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        DEBUG = 'true'
        appversion = ''
        region = 'us-east-1'
        acc_ID = '135808959960'
        project = 'expense'
        environment = 'prod'
        component = 'frontend'

    }
    
    stages {
        stage('Read the application version') {
            steps {
                script{
                    def packageJson = readJSON file: 'readversion/package.json'
                    appversion = packageJson.version
                    echo "app Version: ${appversion}"
                }
            }
        }

        stage("Docker Build Image") {
            steps {
                sh """
                    docker build -t naveenrajoli/backend:${appversion} .
                    docker images
                """
            }
        }

        stage('Push docker image to ECR') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-cred') {
                    sh """                   
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${acc_ID}.dkr.ecr.${region}.amazonaws.com

                        docker build -t ${acc_ID}.dkr.ecr.${region}.amazonaws.com/kdp-${project}-${environment}/${component}:${appversion} .

                        docker images

                        docker push ${acc_ID}.dkr.ecr.${region}.amazonaws.com/kdp-${project}-${environment}/${component}:${appversion}                 

                    """
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-cred') {
                    
                        sh """                        
                            cd helm
                            sed -i "s/IMAGEVERSION/${appversion}/g" values.yaml
                            helm upgrade --install frontend-revchart -n rnk-expense -f values.yaml .
                        """
                    
                }

            }
        }
        

    }
    








    post {
        always{
            echo 'this will run always'
            deleteDir()
        }
        success{
            echo 'this will run on success'
        }
        failure{
            echo 'this will run at failure'
        }
    }
}