pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG',  defaultValue: '', description: 'Docker image tag to deploy')
        string(name: 'NAMESPACE',  defaultValue: 'notes-test', description: 'Kubernetes namespace')
    }

    stages {
        stage('Validate') {
            steps {
                script {
                    if (!params.IMAGE_TAG?.trim()) {
                        error('IMAGE_TAG is required')
                    }
                }
            }
        }

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Deploy') {
            steps {
                withCredentials([
                        file(credentialsId: 'minikube-kubeconfig', variable: 'KUBECONFIG'),
                        string(credentialsId: 'notes-service-postgres-uri', variable: 'POSTGRES_URI')
                ]) {
                    sh """
                        helm upgrade --install notes-service ./helm-notes-service \
                          --kubeconfig \$KUBECONFIG \
                          --namespace ${params.NAMESPACE} \
                          --create-namespace \
                          --set image.repository=ghcr.io/sarka99/notes-service \
                          --set image.tag=${params.IMAGE_TAG} \
                          --set image.pullPolicy=Always \
                          --set "secrets.POSTGRES_URI=\${POSTGRES_URI}" \
                          --wait --timeout 120s
                    """
                }
            }
        }
    }
}