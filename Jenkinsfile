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
                docker network create app-network || true
                docker rm -f backend1 backend2 || true
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                                printf '%s\n' \
                                    'FROM ubuntu:22.04' \
                                    'RUN apt-get update && apt-get install -y nginx' \
                                    'CMD ["nginx", "-g", "daemon off;"]' > Dockerfile.nginx.local
                                docker build -t nginx-local -f Dockerfile.nginx.local .
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                                    nginx-local
                
                                docker exec nginx-lb rm -f /etc/nginx/conf.d/default.conf || true
                                docker cp nginx/default.conf nginx-lb:/etc/nginx/sites-enabled/default
                docker exec nginx-lb nginx -s reload
                                rm -f Dockerfile.nginx.local
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
