pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp:latest'
        IMAGE_TAR = 'myapp.tar'
        APPLICATION_SERVER = 'vagrant@192.168.56.31'
        DEST_PATH = '~'
    }

    stages {
        stage('Checkout') {
            steps {
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

        stage('Create Build Archive') {
            steps {
                script {
                    dir('publish') {
                        sh 'tar czvf ../build.tar.gz .'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Move Dockerfile to the root or specify path if it is located somewhere else
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Save Docker Image') {
            steps {
                script {
                    // Save the Docker image to a tar file
                    sh 'docker save $DOCKER_IMAGE -o $IMAGE_TAR'
                }
            }
        }

        stage('Archive Docker Image') {
            steps {
                archiveArtifacts artifacts: IMAGE_TAR, allowEmptyArchive: true
            }
        }

        stage('Transfer Docker Image') {
            steps {
                script {
                    // Transfer Docker image tar file to the application server
                    sh "scp -i /path/to/ssh/key $IMAGE_TAR $APPLICATION_SERVER:$DEST_PATH"
                }
            }
        }

        stage('Deploy on Application Server') {
            steps {
                script {
                    // Load and run Docker container on the application server
                    sh """
                    ssh -i /path/to/ssh/key $APPLICATION_SERVER '
                        docker load -i $DEST_PATH/$IMAGE_TAR &&
                        docker run -d --name myapp-container -p 80:80 $DOCKER_IMAGE
                    '
                    """
                }
            }
        }
    }
}
