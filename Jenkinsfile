pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        DOCKERHUB_USERNAME = 'srikanth7349'
        BACKEND_IMAGE = "${DOCKERHUB_USERNAME}/employee_backend:v1"
        FRONTEND_IMAGE = "${DOCKERHUB_USERNAME}/employee_frontend:v1"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/akshaygouda1407/Employee_management_system.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('ems-backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('ems-frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $BACKEND_IMAGE ./ems-backend'
                sh 'docker build -t $FRONTEND_IMAGE ./ems-frontend'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKERHUB_CREDENTIALS", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $BACKEND_IMAGE'
                    sh 'docker push $FRONTEND_IMAGE'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker network create employee-network || true

                docker network connect employee-network ems-mysql || true

                docker rm -f backend || true
                docker rm -f frontend || true

                docker run -d --name backend \
                  --network employee-network \
                  -p 8082:8082 \
                  $BACKEND_IMAGE

                docker run -d --name frontend \
                  -p 3001:80 \
                  $FRONTEND_IMAGE
                '''
            }
        }
    }
}
