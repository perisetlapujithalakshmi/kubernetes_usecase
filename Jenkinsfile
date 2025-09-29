pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'pujithaperisetla01'
        IMAGE_NAME     = 'helloworld'
        IMAGE_TAG      = 'latest'
        KUBECONFIG     = "${env.WORKSPACE}/kubeconfig"
    }

    stages {
        stage("Clone Git Repo") {
            steps {
                dir('/data/kubernetes/usecase') {
                    git branch: 'main', url: 'https://github.com/perisetlapujithalakshmi/kubernetes_usecase.git'
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                sh "docker build -t $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG ."
            }
        }

        stage("Push the Image to DockerHub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }

        stage("Scan Image") {
            steps {
                withEnv(["TRIVY_CACHE_DIR=/data/trivy_cache"]) {
                    sh "trivy image $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }

        stage("Create Kind Cluster") {
            steps {
                sh """
                kind create cluster --name mycluster --config /data/kubernetes/usecase/kind-cluster.yaml --kubeconfig $KUBECONFIG
                """
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                sh """
                kubectl --kubeconfig=$KUBECONFIG apply -f /data/kubernetes/usecase/namespace.yaml --validate=false
                kubectl --kubeconfig=$KUBECONFIG apply -f /data/kubernetes/usecase/configmap.yaml
                kubectl --kubeconfig=$KUBECONFIG apply -f /data/kubernetes/usecase/secret.yaml
                kubectl --kubeconfig=$KUBECONFIG apply -f /data/kubernetes/usecase/pvc.yaml
                kubectl --kubeconfig=$KUBECONFIG apply -f /data/kubernetes/usecase/helloworld-deployment.yaml
                kubectl --kubeconfig=$KUBECONFIG apply -f /data/kubernetes/usecase/helloworld-service.yaml
                kubectl --kubeconfig=$KUBECONFIG rollout restart deployment/helloworld-deployment -n pujitha
                """
            }
        }
    }
}
