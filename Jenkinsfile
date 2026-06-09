pipeline {
    agent {
        label 'jenkins-worker'
    }

    environment {
        HARBOR_REGISTRY = '10.178.57.127'
        HARBOR_PROJECT = 'corona-tracker'
        IMAGE_NAME = 'frontend'
        IMAGE_TAG = "${BUILD_NUMBER}"
        HELM_REPO = 'git@github.com:LeoHDuong/corona-tracker-helm.git'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG} \
                        ${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'harbor-credentials',
                    usernameVariable: 'HARBOR_USER',
                    passwordVariable: 'HARBOR_PASS'
                )]) {
                    sh """
                        docker login ${HARBOR_REGISTRY} -u ${HARBOR_USER} -p ${HARBOR_PASS}
                        docker push ${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh """
                    git clone ${HELM_REPO} helm-charts
                    helm upgrade --install frontend helm-charts/frontend \
                        --set image.repository=${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG} \
                        --kubeconfig /var/lib/jenkins/.kube/config
                """
            }
        }
    }

    post {
        always {
            sh 'docker logout ${HARBOR_REGISTRY} || true'
            cleanWs()
        }
        success {
            echo "Frontend deployed successfully! Image tag: ${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
