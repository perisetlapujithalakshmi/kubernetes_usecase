pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'pujithaperisetla01'
        IMAGE_NAME = 'helloworld'
        IMAGE_TAG = 'latest'
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
                sh 'trivy image $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG || true'
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-cred']) {
                    sh '''
                        kubectl apply -f namespace.yaml
                        kubectl apply -f configmap.yaml
                        kubectl apply -f secret.yaml
                        kubectl apply -f pvc.yaml
                        kubectl apply -f helloworld-deployment.yaml
                        kubectl apply -f helloworld-service.yaml

                        # Restart deployment to pull latest image
                        kubectl rollout restart deployment/helloworld-deployment -n pujitha
                    '''
                }
            }
        }
    }
}
