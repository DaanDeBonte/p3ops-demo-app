pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp:latest'
        IMAGE_TAR = 'myapp.tar'
        APPLICATION_SERVER = 'vagrant@192.168.56.31'
        DEST_PATH = '~/'
        DB_CONTAINER_NAME = 'dotnet_db'
        APP_CONTAINER_NAME = 'dotnet_applicatie'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from source control (GitHub, etc.)
                checkout scm
            }
        }

        stage('Build and Save Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t $DOCKER_IMAGE .'
                    // Save Docker image to tarball
                    sh 'docker save $DOCKER_IMAGE -o $IMAGE_TAR'
                }
            }
        }

        stage('Transfer Docker Image') {
            steps {
                script {
                    // Transfer Docker image to the application server
                    sh "scp -i /var/lib/jenkins/.ssh/id_rsa $IMAGE_TAR $APPLICATION_SERVER:$DEST_PATH"
                }
            }
        }

        stage('Deploy on Application Server') {
            steps {
                script {
                    sh """
                    ssh -i /var/lib/jenkins/.ssh/id_rsa $APPLICATION_SERVER '
                        # Load Docker image
                        sudo docker load -i $DEST_PATH/$IMAGE_TAR

                        # Stop and remove any existing containers
                        sudo docker stop $APP_CONTAINER_NAME || true
                        sudo docker rm $APP_CONTAINER_NAME || true
                        sudo docker stop $DB_CONTAINER_NAME || true
                        sudo docker rm $DB_CONTAINER_NAME || true

                        # Run the SQL Server container
                        sudo docker run -d \
                            --name $DB_CONTAINER_NAME \
                            --network myapp-network \
                            -e ACCEPT_EULA=Y \
                            -e MSSQL_SA_PASSWORD=example_123 \
                            -p 1433:1433 \
                            mcr.microsoft.com/mssql/server:2019-latest

                        # Run the application container
                        sudo docker run -d \
                            --name $APP_CONTAINER_NAME \
                            --network myapp-network \
                            -p 8081:80 \
                            -e DOTNET_ConnectionStrings__SqlDatabase="Server=$DB_CONTAINER_NAME;Database=myDataBase;User ID=sa;Password=example_123;" \
                            $DOCKER_IMAGE
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
