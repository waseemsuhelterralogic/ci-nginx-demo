pipeline {
    agent { label 'rke2' }

    environment {
        DOCKERHUB_USER = '<your-dockerhub-username>'
        KUBE_NAMESPACE = 'ci-demo'
        K8S_SERVER = 'https://localhost:16443'
        K8S_TOKEN = 'eyJhbGciOiJSUzI1NiIsImtpZCI6IlNSN1ZvM3UtSEFRYTRjT2lobFQxblR6V21uNks4dnd3ZU5YMVFBSVlaOWsifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJya2UyIl0sImV4cCI6MTc2NTk3NDU1NCwiaWF0IjoxNzY1OTcwOTU0LCJpc3MiOiJodHRwczovL2t1YmVybmV0ZXMuZGVmYXVsdC5zdmMuY2x1c3Rlci5sb2NhbCIsImp0aSI6ImUzMWIyODMyLTg2OTctNDg0Mi04NmQ3LThkNzE0OGVhZTljNyIsImt1YmVybmV0ZXMuaW8iOnsibmFtZXNwYWNlIjoiY2ktZGVtbyIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJqZW5raW5zLXNhIiwidWlkIjoiZDdjZTRlMDktZGRkNS00ZjYyLTg2OWEtODMyZTM0NjczOGUyIn19LCJuYmYiOjE3NjU5NzA5NTQsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpjaS1kZW1vOmplbmtpbnMtc2EifQ.h2_hCw56WdDWHrU3gL4BAVMrC5tUzzDk73RD2x1nndPV2e13ktmMVJ3iYI23kVrROpS1t3rdZfPmh3O1WcqitUfuEH2dvqiwsmWvuSq1brAMdbqbSAdkRTcsoPz1ou1QIUt_KYRLa8GxLAYmrhxV5uiGSKRY9N2Lv3OMU0UbX5-5AZTdjWK9O26SCNPR3KjfGrf-NtkZrKNRj2fKPSev09aRvfeClWEpKaxYzy2RJRWJ5785WdWyYuBRVUgb5Iy0LiSqfnAC_-av35ZK5wzOOCgI2qHHkcpfMCUU1ObFqG7pWNjiU5bduSsM7fiSK8VVmv2K_GR7PAKyJQUvTDRUFA'
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
                        echo "$DH_PASS" | podman login docker.io -u "$DH_USER" --password-stdin
                        podman tag ci-nginx-demo:latest docker.io/$DH_USER/ci-nginx-demo:latest
                        podman push docker.io/$DH_USER/ci-nginx-demo:latest
                    '''
                }
            }
        }

        stage('Create Namespace') {
            steps {
                sh '''
                kubectl --server=$K8S_SERVER \
                --token=$K8S_TOKEN \
                --insecure-skip-tls-verify \
                create namespace $KUBE_NAMESPACE --dry-run=client -o yaml | \
                kubectl --server=$K8S_SERVER \
                --token=$K8S_TOKEN \
                --insecure-skip-tls-verify \
                apply --validate=false -f -
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl --server=$K8S_SERVER \
                    --token=$K8S_TOKEN \
                    --insecure-skip-tls-verify \
                    apply -f /home/jenkins-agent/ci-nginx-demo/nginx-deployment.yaml -n ci-demo

                    kubectl --server=$K8S_SERVER \
                    --token=$K8S_TOKEN \
                    --insecure-skip-tls-verify \
                    apply -f /home/jenkins-agent/ci-nginx-demo/nginx-service.yaml -n ci-demo
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}

