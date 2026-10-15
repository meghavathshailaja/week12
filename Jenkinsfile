
pipeline {
    agent any

    environment {
        PYTHON = 'C:\\Users\\megav\\AppData\\Local\\Programs\\Python\\Python313\\python.exe'
        DOCKER_IMAGE = 'sailayasample/seleniumtestimage:v1'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo 'GitHub repository checked out successfully.'
            }
        }

        stage('Check Python and Install Dependencies') {
            steps {
                echo 'Checking Python version'
                bat '"%PYTHON%" --version'

                echo 'Installing required Python packages'
                bat '"%PYTHON%" -m pip install -r requirements.txt'
            }
        }

        stage('Run Selenium Tests') {
            steps {
                echo 'Starting Flask application'

                bat 'start "FlaskApp" /B "%PYTHON%" app.py'

                echo 'Waiting for application startup'
                bat 'ping 127.0.0.1 -n 6 > nul'

                echo 'Running registration application tests'
                bat '"%PYTHON%" -m pytest -v test/testregistrationapp.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                bat 'docker build -t %DOCKER_IMAGE% -f dockerfile .'
            }
        }

        stage('Docker Login and Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    echo 'Logging in to Docker Hub'

                    bat 'powershell -NoProfile -NonInteractive -Command "$env:DOCKER_PASS | docker login --username $env:DOCKER_USER --password-stdin"'

                    echo 'Pushing Docker image'
                    bat 'docker push %DOCKER_IMAGE%'

                    bat 'docker logout'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes'

                bat 'kubectl apply -f deployment.yaml --validate=false'
                bat 'kubectl apply -f service.yaml'

                echo 'Checking Kubernetes resources'
                bat 'kubectl get deployments'
                bat 'kubectl get services'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }

        success {
            echo 'SUCCESS: All pipeline stages completed successfully!'
        }

        failure {
            echo 'FAILED: Check Console Output for the stage that failed.'
        }
    }
}

