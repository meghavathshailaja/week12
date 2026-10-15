
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
            }
        }

        stage('Python Check and Dependencies') {
            steps {
                echo 'Checking Python installation'
                bat '"%PYTHON%" --version'
                bat '"%PYTHON%" -m pip install -r requirements.txt'
            }
        }

        stage('Run Selenium Tests with pytest') {
            steps {
                echo 'Starting application'
                bat 'start "FlaskApp" /B "%PYTHON%" app.py'
                bat 'ping 127.0.0.1 -n 6 > nul'

                echo 'Running Selenium tests'
                bat '"%PYTHON%" -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image'
                bat 'docker build -t %DOCKER_IMAGE% .'
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
                    bat 'powershell -NoProfile -NonInteractive -Command "$env:DOCKER_PASS | docker login --username $env:DOCKER_USER --password-stdin"'
                    bat 'docker push %DOCKER_IMAGE%'
                    bat 'docker logout'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml --validate=false'
                bat 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
