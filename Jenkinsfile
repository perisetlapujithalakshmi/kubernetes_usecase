pipeline{
  agent any
  environment{
    DOCKERHUB_USER = pujithaperisetla01
    IMAGE_NAME = helloworld
    IMAGE_TAG = latest
  }
  stages{
    stage("clone git repo'){
      steps{
        git branch: 'main',url : 'https://github.com/perisetlapujithalakshmi/kubernetes_usecase.git'
      }
    }
    stage("Build docker image"){
      steps{
        sh 'docker build -t $DOCKERHUB_USER/$IMAGE_NAME/$IMAGE_TAG .'
      }
    }
    stage("push the image to dockerhub"){
      steps{
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG'
        }
    }
    stage("Scan image"){
      steps{
        sh 'trivy image $DOCKERHUB_USER/$IMAGE_NAME/$IMAGE_TAG || true'
      }
    }
    stage("Deploy to kubernetes"){
      steps{
        withKubeConfig([credentialsId: 'kubeconfig-cred', serverUrl: 'https://<your-cluster-endpoint>']){
          sh '''
          kubectl apply -f namespace.yaml
          kubectl apply -f configmap.yaml
          kubectl apply -f secret.yaml
          kubectl apply -f pvc.yaml
          kubectl apply -f helloworld-deployment.yaml
          kubectl apply -f helloworld-service.yaml
          '''
        }
      }
    }
    }
          }
