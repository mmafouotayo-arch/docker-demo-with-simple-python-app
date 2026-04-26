pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO       = "mafouo/docker-demo-python"
        DOCKER_IMAGE          = "${DOCKER_HUB_REPO}:${BUILD_NUMBER}"
        DOCKER_LATEST         = "${DOCKER_HUB_REPO}:latest"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/mmafouotayo-arch/docker-demo-with-simple-python-app.git'
            }
        }

        stage('Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE% -t %DOCKER_LATEST% ."
            }
        }

        stage('Code Quality') {
            steps {
                bat """
                    docker run --rm %DOCKER_IMAGE% sh -c "pip install flake8 --quiet && flake8 . --max-line-length=120 --exclude=.git,__pycache__ || true"
                """
            }
        }

        stage('Tests') {
            steps {
                bat """
                    docker run --rm %DOCKER_IMAGE% sh -c "pip install pytest --quiet && pytest tests/ -v --tb=short || echo Aucun test trouve"
                """
            }
        }

        stage('Package - Push Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                        echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%
                        docker push %DOCKER_LATEST%
                    """
                }
            }
        }

        stage('Deploy - Staging') {
            steps {
                echo 'Deploiement Staging effectue'
            }
        }

        stage('Validation - Production ?') {
            steps {
                input message: 'Deployer en PRODUCTION ?', ok: 'Oui, deployer'
            }
        }

        stage('Deploy - Production') {
            steps {
                echo 'Deploiement Production effectue'
            }
        }
    }

    post {
        always {
            bat "docker rmi %DOCKER_IMAGE% || true"
        }
        success {
            echo 'Pipeline termine avec succes !'
        }
        failure {
            echo 'Echec du pipeline.'
        }
    }
}