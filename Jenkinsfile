pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'madycloudmd/react-demo-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/MaDycloud-MD/app_for_jenkins.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                      echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                      docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}
