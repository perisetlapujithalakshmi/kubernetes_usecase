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
                dir('/data/kubernetes/usecase') {
                    git branch: 'main', url: 'https://github.com/perisetlapujithalakshmi/kubernetes_usecase.git'
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                dir('/data/kubernetes/usecase') { // Make sure Docker build runs in correct directory
                    sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage("Push the Image to DockerHub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage("Scan Image") {
            steps {
                withEnv(["TRIVY_CACHE_DIR=/data/trivy_cache"]) {
                    sh "trivy image ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes via Helm') {
            steps {
                withEnv(["KUBECONFIG=/home/ubuntu/.kube/config"]) {
                    sh """
                        cd /data/kubernetes/helm/helloworld
                        helm upgrade --install helloworld . \
                            --set image.repository=${DOCKERHUB_USER}/${IMAGE_NAME} \
                            --set image.tag=${IMAGE_TAG}
                    """
                }
            }
        }
    }
}
