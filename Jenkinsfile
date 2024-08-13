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

        stage('Build and Start Containers') {
            steps {
                script {
                    // Start the Docker Compose services
                    sh 'docker-compose up -d'

                    // Wait for services to be up and running
                    waitForSqlService('db')

                    // Build your application
                    sh 'dotnet restore src/Server/Server.csproj'
                    sh 'dotnet build src/Server/Server.csproj --os linux'
                    sh 'dotnet publish src/Server/Server.csproj -c Release -o publish'
                }
            }
        }

        stage('Create Docker Image') {
            steps {
                script {
                    // Use the existing Docker Compose environment to build the image
                    sh 'docker-compose exec $DOCKER_IMAGE docker build -t $DOCKER_IMAGE .'
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
                script {
                    sh "scp -i /var/lib/jenkins/.ssh/id_rsa $IMAGE_TAR $APPLICATION_SERVER:$DEST_PATH"
                }
            }
        }

        stage('Deploy on Application Server') {
            steps {
                script {
                    sh """
                    ssh -i /var/lib/jenkins/.ssh/id_rsa $APPLICATION_SERVER '
                        sudo docker load -i $DEST_PATH/$IMAGE_TAR &&
                        sudo docker stop myapp-container || true &&
                        sudo docker rm myapp-container || true &&
                        sudo docker run -d --name myapp-container -p 80:80 $DOCKER_IMAGE
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up - stop and remove containers
            sh 'docker-compose down'
            cleanWs()
        }
    }
}

def waitForSqlService(serviceName) {
    timeout(time: 5, unit: 'MINUTES') {
        waitUntil {
            return sh(script: "docker-compose ps -q $serviceName | xargs docker inspect --format '{{.State.Status}}'", returnStatus: true) == 0
        }
    }
}

