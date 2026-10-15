
pipeline {
    agent any

    stages {
        stage('Run Selenium Tests with pytest') {
            steps {
                echo "Running Selenium Tests using pytest"

                bat 'py -m pip install -r requirements.txt'
                bat 'start /B py app.py'
                bat 'ping 127.0.0.1 -n 5 > nul'
                bat 'py -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image"
                bat 'docker build -t seleniumdemoapp:v1 .'
            }
        }

        stage('Docker Login') {
            steps {
                echo "Logging in to Docker Hub"

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat 'echo %DOCKER_PASS%| docker login -u "%DOCKER_USER%" --password-stdin'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "Pushing Docker Image to Docker Hub"

                bat 'docker tag seleniumdemoapp:v1 sailayasample/seleniumtestimage:v1'
                bat 'docker push sailayasample/seleniumtestimage:v1'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying Application to Kubernetes"

                bat 'kubectl apply -f deployment.yaml --validate=false'
                bat 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
