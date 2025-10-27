pipeline {
    agent any

    environment {
        IMAGE_NAME = "endsem-app"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        stage('Build with Maven (Java 21)') {
            steps {
                echo "Running Maven build..."
                bat 'mvn -B clean package'
            }
        }

        stage('Test Run') {
            steps {
                echo "Testing the JAR output..."
                bat 'java -jar target\\endsem-app-1.0-SNAPSHOT.jar'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image..."
                bat """
                    docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                    docker images | findstr %IMAGE_NAME%
                """
            }
        }

        stage('Docker Run (Smoke Test)') {
            steps {
                echo "Running Docker container..."
                bat """
                    docker rm -f endsem-run || echo no-container
                    docker run --name endsem-run -d -p 8080:8080 %IMAGE_NAME%:%IMAGE_TAG%
                    timeout /t 5 >nul
                    docker logs endsem-run
                    docker rm -f endsem-run
                """
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
