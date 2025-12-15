pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'tahersahbi/students-management'
        DOCKER_TAG   = "${BUILD_NUMBER}"
        SONAR_HOST   = 'http://host.docker.internal:9000'
        KUBECONFIG   = '/var/lib/jenkins/.kube/config'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo '====== Checking out code from GitHub ======'
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/Taher387/ProjetStudentsManagement-DevOps.git'
            }
        }
        
        stage('Build with Maven') {
            steps {
                echo '====== Building application with Maven ======'
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '====== Running SonarQube analysis ======'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=students-management \
                          -Dsonar.projectName="Students Management" \
                          -Dsonar.host.url=${SONAR_HOST} \
                          -Dsonar.token=\$SONAR_TOKEN
                    """
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '====== Building Docker image ======'
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }
        
        stage('Push Docker Image') {
            steps {
                echo '====== Pushing Docker image to DockerHub ======'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                echo '====== Deploying to Kubernetes ======'
                sh """
                    kubectl set image deployment/spring-app \
                      spring-app=${DOCKER_IMAGE}:${DOCKER_TAG} \
                      -n devops
                    kubectl rollout status deployment/spring-app -n devops
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo '====== Verifying deployment ======'
                sh """
                    kubectl get pods -n devops
                    kubectl get svc spring-service -n devops
                """
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            echo "🚀 Image deployed: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo '❌ Pipeline failed. Check Jenkins logs.'
        }
        always {
            echo '🧹 Cleaning workspace...'
            cleanWs()
        }
    }
}
