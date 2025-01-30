pipeline {
    agent any
    environment {
        FRONTEND_IMAGE = 'frontend-app'
        BACKEND_IMAGE = 'backend-app'
        DB_CONTAINER = 'postgres-db'
        NETWORK_NAME = 'my-network'
        VOLUME_NAME = 'pg_data'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '3f601ce9-7463-4bde-bf20-ca21df940366', url: 'https://github.com/teja94411/Docker-Containerization.git']])
                bat 'dir'  // List the contents of the workspace to verify repository structure
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker build -t ${FRONTEND_IMAGE} -f Greenstone-Docker/Dockerfile ./frontend  // Build the frontend image
                    '''
                }
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker build -t ${BACKEND_IMAGE} -f Greenstone-Docker/Dockerfile ./backend  // Build the backend image
                    '''
                }
            }
        }

        stage('Setup Docker Network and Volume') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker volume create ${VOLUME_NAME}  // Create a Docker volume for PostgreSQL
                    docker network create ${NETWORK_NAME}  // Create a Docker network
                    '''
                }
            }
        }

        stage('Run PostgreSQL') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker run -d --network ${NETWORK_NAME} --name ${DB_CONTAINER} -e POSTGRES_DB=testdb -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -v ${VOLUME_NAME}:/var/lib/postgresql/data postgres:13
                    '''
                }
            }
        }

        stage('Run Backend') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker run -d --network ${NETWORK_NAME} --name backend -e DB_HOST=${DB_CONTAINER} -e DB_NAME=testdb -e DB_USER=user -e DB_PASSWORD=password -p 5000:5000 ${BACKEND_IMAGE}
                    '''
                }
            }
        }

        stage('Run Frontend') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker run -d --network ${NETWORK_NAME} --name frontend -p 3000:3000 ${FRONTEND_IMAGE}
                    '''
                }
            }
        }

        stage('Run JUnit Tests') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker exec backend python3 -m unittest discover tests/  // Run JUnit tests inside the backend container
                    '''
                }
            }
        }

        stage('Teardown') {
            steps {
                script {
                    bat '''
                    cd Greenstone-Docker  // Change to the Greenstone-Docker directory
                    docker stop frontend ${DB_CONTAINER}  // Stop frontend and PostgreSQL containers
                    docker rm frontend backend ${DB_CONTAINER}  // Remove the containers
                    docker network rm ${NETWORK_NAME}  // Remove the Docker network
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully'
        }
        failure {
            echo 'Pipeline failed. Please check logs'
        }
    }
}
