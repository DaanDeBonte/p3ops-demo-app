pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp:latest'
        IMAGE_TAR = 'myapp.tar'
        APPLICATION_SERVER = 'vagrant@192.168.56.31'
        DEST_PATH = '~/'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from source control (GitHub, etc.)
                checkout scm
            }
        }

        stage('Build .NET Application') {
            steps {
                script {
                    sh 'dotnet restore src/Server/Server.csproj'
                    sh 'dotnet build src/Server/Server.csproj --os linux'
                    sh 'dotnet publish src/Server/Server.csproj -c Release -o publish'
                }
            }
        }

        stage('Create Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Save Docker Image') {
            steps {
                script {
                    sh 'docker save $DOCKER_IMAGE -o $IMAGE_TAR'
                }
            }
        }

        stage('Transfer Docker Image') {
            steps {
                sshagent(credentials: ['jenkinsssh']) {
                    script {
                        sh "scp $IMAGE_TAR $APPLICATION_SERVER:$DEST_PATH"
                    }
                }
            }
        }

        stage('Deploy on Application Server') {
            steps {
                sshagent(credentials: ['jenkinsssh']) {
                    script {
                        sh """
                        ssh $APPLICATION_SERVER '
                            docker load -i $DEST_PATH/$IMAGE_TAR &&
                            docker stop myapp-container || true &&
                            docker rm myapp-container || true &&
                            docker run -d --name myapp-container -p 80:80 $DOCKER_IMAGE
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
