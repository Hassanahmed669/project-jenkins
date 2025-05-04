pipeline {
    agent any
    environment {
      DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
      IMAGE_NAME = "hassanhbahmed/sample-app"
      IMAGE_TAG = "${BUILD_NUMBER}"
    }    
    stages {
        stage('Building Docker Image...'){
            steps {
                script {
                    echo "Building the docker image of our java maven applicaiton"
                    sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
                }
            }
        }
        stage('Test Docker Image') {
          steps {
            sh '''
            docker run -d --name test_container $IMAGE_NAME:IMAGE_TAG
            sleep 5
            docker stop test_container
            docker rm test_container
            '''
          }
        }        
        stage('Push to DockerHub') {
          steps {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
              sh '''
                echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                docker push $IMAGE_NAME:IMAGE_TAG
              '''
                }
            }
        }
        stage('Deploy to Staging via Ansible') {
          steps {
            sh 'ansible-playbook -i ansible/inventory.ini ansible/deploy_k8s.yaml --extra-vars "image_tag=$IMAGE_TAG'
            }
        }
    }
}
