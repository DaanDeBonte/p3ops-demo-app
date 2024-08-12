pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp:latest'
        IMAGE_TAR = 'myapp.tar'
        APPLICATION_SERVER = 'vagrant@192.168.56.31'
        DEST_PATH = '~/'
        SSH_KEY_PATH = '/home/vagrant/.ssh/id_rsa.pub'  // Path to your SSH key on Jenkins server
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
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }

        stage('Save Docker Image') {
            steps {
                script {
                    sh "docker save $DOCKER_IMAGE -o $IMAGE_TAR"
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
                    sh "sudo scp -i $SSH_KEY_PATH $IMAGE_TAR $APPLICATION_SERVER:$DEST_PATH"
                }
            }
        }

        stage('Deploy on Application Server') {
            steps {
                script {
                    sh """
                    ssh -i $SSH_KEY_PATH $APPLICATION_SERVER '
                        docker load -i $DEST_PATH/$IMAGE_TAR &&
                        docker run -d --name myapp-container -p 80:80 $DOCKER_IMAGE
                    '
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()  // Clean workspace after the build is finished
        }
    }
}
