pipeline {
    agent any

    environment {
        REGISTRY = "hastein10"
        IMAGE_MOVIE = "${REGISTRY}/movie-service"
        IMAGE_CAST = "${REGISTRY}/cast-service"
        CHART_PATH = "./charts"
        KUBE_NAMESPACE = "devops-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test Docker Access') {
            steps {
                sh 'docker info'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $IMAGE_MOVIE:latest ./movie-service'
                sh 'docker build -t $IMAGE_CAST:latest ./cast-service'
            }
        }

        stage('Push Docker Images') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([
                    string(credentialsId: 'dockerhub-username-id', variable: 'DOCKERHUB_USERNAME'),
                    string(credentialsId: 'dockerhub-password-id', variable: 'DOCKERHUB_PASSWORD')
                ]) {
                    sh """
                        echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                        docker push $IMAGE_MOVIE:latest
                        docker push $IMAGE_CAST:latest
                    """
                }
            }
        }

        stage('Manual Approval for Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
            }
        }

        stage('Deploy to Production (Helm)') {
            when {
                branch 'main'
            }
            steps {
                sh "helm upgrade --install devops-prod ${CHART_PATH} --namespace prod --create-namespace -f ${CHART_PATH}/values.yaml"
            }
        }
    }

    post {
        success {
            echo 'Pipeline success'
        }
        failure {
            echo 'Pipeline failed'
        }
        always {
            cleanWs()
        }
    }
}
