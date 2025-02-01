pipeline {
    agent any
    environment {
        FRONTEND_IMAGE = 'nginx:latest'  // Use Nginx image for serving static files
        BACKEND_IMAGE = 'python:3.9-slim'  // Use Python image to run the Flask app
        DB_CONTAINER = 'postgres-db'
        NETWORK_NAME = 'my-network'
        VOLUME_NAME = 'pg_data'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '3f601ce9-7463-4bde-bf20-ca21df940366', url: 'https://github.com/teja94411/Docker-Containerization.git']])
                bat 'dir'  // Listing the directory contents to verify workspace structure
            }
        }
        stage('Setup Docker Network and Volume') {
            steps {
                script {
                    // Create a Docker volume for PostgreSQL
                    bat "docker volume create %VOLUME_NAME%"
                    // Verify volume creation
                    bat "docker volume ls"

                    // Create a Docker network
                    bat "docker network create %NETWORK_NAME%"
                    // Verify network creation
                    bat "docker network ls"
                }
            }
        }
        stage('Run PostgreSQL') {
    steps {
        script {
            // Run PostgreSQL container with the created volume
            bat '''
            docker run -d ^
                --network %NETWORK_NAME% ^
                --name %DB_CONTAINER% ^
                -e POSTGRES_DB=testdb ^
                -e POSTGRES_USER=user ^
                -e POSTGRES_PASSWORD=password ^
                -v %VOLUME_NAME%:/var/lib/postgresql/data ^
                postgres:13
            '''
            // Wait for 10 seconds to ensure PostgreSQL is fully initialized
            bat "ping -n 10 127.0.0.1 > nul"  // Wait 10 seconds using ping as a workaround for timeout
        }
    }
}

        stage('Run Backend') {
    steps {
        script {
            // Run the Backend (Flask) app using Python image
            bat '''
            docker run -d ^
                --network %NETWORK_NAME% ^
                --name backend ^
                -e DB_HOST=%DB_CONTAINER% ^
                -e DB_NAME=testdb ^
                -e DB_USER=user ^
                -e DB_PASSWORD=password ^
                -p 5000:5000 ^
                -v %WORKSPACE%\\backend:/app ^
                %BACKEND_IMAGE% bash -c "pip install -r /app/requirements.txt && python /app/app.py"
            '''
        }
    }
}



        stage('Run Frontend') {
            steps {
                script {
                    // Run the Frontend using the Nginx image to serve static files
                    bat '''
                    docker run -d ^
                        --network %NETWORK_NAME% ^
                        --name frontend ^
                        -v %WORKSPACE%\\frontend:/usr/share/nginx/html:ro ^
                        -p 80:80 ^
                        %FRONTEND_IMAGE%
                    '''
                }
            }
        }
        stage('Teardown') {
            steps {
                script {
                    // Stop and remove containers and network
                    bat '''
                    docker stop frontend backend %DB_CONTAINER%
                    docker rm frontend backend %DB_CONTAINER%
                    docker network rm %NETWORK_NAME%
                    docker volume rm %VOLUME_NAME%
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
