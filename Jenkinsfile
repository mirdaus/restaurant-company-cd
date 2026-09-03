pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:${env.PATH}"

        AWS_REGION = 'us-east-2'
        EKS_CLUSTER = 'new-eks'
        NAMESPACE = 'restaurant-prod'
}
    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify AWS') {
            steps {
                sh '''
                    aws --version
                    aws sts get-caller-identity
                '''
            }
        }

        stage('Connect to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                        --region ${AWS_REGION} \
                        --name ${EKS_CLUSTER}

                    kubectl cluster-info
                    kubectl get nodes
                '''
            }
        }

        stage('Deploy Namespace') {
            steps {
                sh '''
                    kubectl apply -f namespace.yaml
                '''
            }
        }

        stage('Deploy Kubernetes Resources') {
            steps {
                sh '''
                    kubectl apply -f serviceaccount.yaml
                    kubectl apply -f configmap.yaml
                    kubectl apply -f service.yaml
                    kubectl apply -f deployment.yaml
                    kubectl apply -f hpa.yaml
                    kubectl apply -f pdb.yaml
                    kubectl apply -f networkpolicy.yaml
                    kubectl apply -f ingress.yaml
                '''
            }
        }

        stage('Restart Deployment') {
            steps {
                sh '''
                    kubectl rollout restart deployment/restaurant-company \
                        -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/restaurant-company \
                        -n ${NAMESPACE} \
                        --timeout=180s

                    kubectl get pods -n ${NAMESPACE}
                    kubectl get svc -n ${NAMESPACE}
                    kubectl get ingress -n ${NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to EKS completed successfully!'
        }

        failure {
            echo '❌ Deployment to EKS failed. Check Jenkins console output.'
        }
    }
}
