pipeline {
  agent {
    node {
      label 'python'
    }
  }
  stages {
    stage('Checkout') {
         steps {
           container('dind') {
            echo "Checking out Code Templates"
            git branch: 'master', changelog: false, url: 'https://github.com/Mukesh-50/frontend_node.git'
         }
        }
      }
    stage('Building Docker Image') {
        steps{
            container('dind') {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubpasswd', usernameVariable: 'dockerhubuser')]) { 
                sh '''
                pwd
                ls
                # cd frontend_node
                docker build -t frontend:v1 .
                docker tag frontend:v1 nicksrj/frontend:v1
                docker login -u $dockerhubuser -p $dockerhubpasswd
                docker push nicksrj/frontend:v1
                '''
                }
             }
           }
        }
    stage('fetching kubeconfig') {
        steps {
            container('dind') {
                withCredentials([usernamePassword(credentialsId: 'awscred', passwordVariable: 'secretkey', usernameVariable: 'accesskey')]) {
                sh '''
                /usr/local/bin/aws --version version
                /usr/local/bin/aws --version configure set aws_access_key_id=$accesskey
                /usr/local/bin/aws --version configure set aws_secret_access_key=$secretkey
                /usr/local/bin/aws --version s3 ls
                /usr/local/bin/aws --version eks update-kubeconfig --name celestials
                kubectl version
                '''
                input message: 'Shall we deploy to Prod', ok: 'Deploy'
                  }
                }
            }
        }
    stage('Deploy App') {
      steps {
        container('dind') {
          sh '''
          kubectl get deployment -n prodcatalog-ns
          export REPOSITORY_URI=nicksrj/frontend:v1
          envsubst < kubespec/frontendnodekubedeploy.yaml | kubectl apply -f -
          ATTEMPTS=0
              DEPLOYMENT_NAME=frontend-node
              ROLLOUT_STATUS_CMD="kubectl rollout status deployment/$DEPLOYMENT_NAME -n prodcatalog-ns "
              until $ROLLOUT_STATUS_CMD || [ $ATTEMPTS -eq 60 ]; do
                 $ROLLOUT_STATUS_CMD
                 ATTEMPTS=$((attempts + 1))
                 sleep 10
              done
              sleep 5
          
          '''
        }
      }
    }
  }
}
