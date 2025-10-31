pipeline {
    agent { label 'rushi' }

    environment {
        DOCKER_REGISTRY = "rushikeshdoc"
        BACKEND_IMAGE = "backend:latest"
        FRONTEND_IMAGE = "frontend:latest"
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/rushikeshgoal/hrmsdemo.git', branch: 'main'
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('Backend_hrms') {
                    sh "docker build -t $DOCKER_REGISTRY/$BACKEND_IMAGE ."
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t $DOCKER_REGISTRY/$FRONTEND_IMAGE ."
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
                sh "docker push $DOCKER_REGISTRY/$BACKEND_IMAGE"
                sh "docker push $DOCKER_REGISTRY/$FRONTEND_IMAGE"
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh "docker compose -f $COMPOSE_FILE down"
                sh "docker system prune -f"
                sh "docker compose -f $COMPOSE_FILE up -d --build"
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}  
