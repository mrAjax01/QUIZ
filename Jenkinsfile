pipeline {
    agent any

    environment {
        IMAGE_NAME = "quiz-app"
        TAG        = "latest"
    }

    stages {
        stage('1. Code download') {
            steps {
                echo 'GitHub downloading ......'
                sh 'ls -la'
            }
        }

        stage('2. Build  Docker (Laravel + React)') {
            steps {
                echo 'Start Multi-stage. Node.js build React,  PHP 8.5 make backend'
                sh 'docker build -f docker/php/Dockerfile -t ${IMAGE_NAME}:${TAG} .'
            }
        }

        stage('3. Deploy in Kubernetes (K3s)') {
            steps {
                echo 'manifests in kube'
                sh 'kubectl apply -f k8s/app/deployment.yaml'

                echo 'pods update'
                sh 'kubectl rollout restart deployment/quiz-app'
            }
        }

        stage('4. Laravel migrations') {
            steps {
                echo 'wait for pod'
                sh 'kubectl rollout status deployment/quiz-app --timeout=60s'

                echo 'start migrations in pod'
                sh '''
                    POD_NAME=$(kubectl get pods -l app=quiz-app -o jsonpath="{.items[0].metadata.name}")
                    kubectl exec $POD_NAME -- php artisan migrate --force
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline done. OK.'
        }
        failure {
            echo 'Deploy error. see stages logs'
        }
    }
}