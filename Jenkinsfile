pipeline {
    agent any

    environment {
        IMAGE_NAME = "madycloudmd/app_for_jenkins"
    }

    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/MaDycloud-MD/app_for_jenkins.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'echo "No tests yet!"'
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
