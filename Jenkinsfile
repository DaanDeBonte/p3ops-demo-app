pipeline {
    agent any

    stages {
        stage('Build and Start Containers') {
            steps {
                script {
                    // Start the Docker Compose services
                    sh 'docker-compose up -d'

                    // Wait for services to be up and running
                    waitForSqlService('db')

                    // Build your application
                    sh 'docker-compose exec your-app docker build -t your-app .'
                }
            }
        }
    }

    post {
        always {
            // Clean up - stop and remove containers
            sh 'docker-compose down'
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
