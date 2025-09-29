pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'pujithaperisetla01'
        IMAGE_NAME     = 'helloworld'
        IMAGE_TAG      = 'latest'
    }

    stages {
        stage("Clone Git Repo") {
            steps {
               dir('/data/kubernetes/usecase'){
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
                    sh 'docker push $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG'
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

        stage("Deploy to Kubernetes") {
    steps {
        withEnv(["KUBECONFIG=/var/jenkins_home/.kube/config"]) {
            sh '''
                kubectl  apply -f /data/kubernetes/usecase/namespace.yaml --validate=false
                kubectl  apply -f /data/kubernetes/usecase/configmap.yaml
                kubectl  apply -f /data/kubernetes/usecase/secret.yaml
                kubectl  apply -f /data/kubernetes/usecase/pvc.yaml
                kubectl  apply -f /data/kubernetes/usecase/helloworld-deployment.yaml
                kubectl  apply -f /data/kubernetes/usecase/helloworld-service.yaml

                kubectl --insecure-skip-tls-verify rollout restart deployment/helloworld-deployment -n pujitha
            '''
        }
    }
}

    }
}
