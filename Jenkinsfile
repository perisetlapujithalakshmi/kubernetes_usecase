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
                git branch: 'main', url: 'https://github.com/perisetlapujithalakshmi/kubernetes_usecase.git'
            }
        }

        stage("Build Docker Image") {
            steps {
                sh 'docker build -t $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG .'
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
                withEnv(["KUBECONFIG=/root/.kube/config"]) {
                    sh "kubectl get pods -A"
            


                    sh '''
                        kubectl apply -f namespace.yaml --validate=false
                        kubectl apply -f configmap.yaml
                        kubectl apply -f secret.yaml
                        kubectl apply -f pvc.yaml
                        kubectl apply -f helloworld-deployment.yaml
                        kubectl apply -f helloworld-service.yaml

                        kubectl rollout restart deployment/helloworld-deployment -n pujitha
                    '''
                }
            }
        }
    }
}
