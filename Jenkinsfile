pipeline {
    agent { label 'rke2' }

    environment {
        DOCKERHUB_USER = '<your-dockerhub-username>'
        KUBE_NAMESPACE = 'ci-demo'
    }

    stages {


        stage('Build Image') {
            steps {
                sh 'podman build -t ci-nginx-demo:latest .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh '''
                        podman login docker.io -u $DH_USER -p $DH_PASS
                        podman tag ci-nginx-demo:latest docker.io/$DH_USER/ci-nginx-demo:latest
                        podman push docker.io/$DH_USER/ci-nginx-demo:latest
                    '''
                }
            }
        }

        stage('Create Namespace') {
            steps {
                sh "kubectl create namespace ${KUBE_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    export PATH=$PATH:/var/lib/rancher/rke2/bin
                    kubectl apply -f nginx-deployment.yaml -n ${KUBE_NAMESPACE}
                    kubectl apply -f nginx-service.yaml -n ${KUBE_NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}

