pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create network if not exists
                docker network inspect app-network >/dev/null 2>&1 || docker network create app-network

                # Remove old containers if they exist
                docker rm -f backend1 backend2 || true

                # Start backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # Wait for containers to fully initialize
                sleep 5

                # Verify they are running
                docker ps
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove old nginx container
                docker rm -f nginx-lb || true

                # Start nginx
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 8081:80 \
                  nginx

                # Wait before copying config
                sleep 2

                # Copy updated nginx config
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Reload nginx
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
