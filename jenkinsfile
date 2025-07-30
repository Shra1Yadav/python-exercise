pipeline {
    agent any

    environment {
        IMAGE_NAME = 'shra1995yadav/python-app:latest'
        REMOTE_HOST = '13.201.95.248'
        REMOTE_USER = 'ubuntu'
        DOCKER_CREDS = credentials('dockerhub-creds')
        REMOTE_APP_DIR = '/var/lib/jenkins/workspace/python-app-declarative-1'
        GIT_REPO = 'https://github.com/Shra1Yadav/python-exercise.git'
    }

    stages {
        stage('Clone Git Repository') {
            steps {
                echo '>>> Cloning Git Repository'
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'github',
                        url: "${GIT_REPO}"
                    ]]
                )
            }
        }

        stage('Docker Login') {
            steps {
                echo '>>> Logging into Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
                    sh '''
                        echo "$DOCKER_HUB_PASS" | docker login -u "$DOCKER_HUB_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo '>>> Building Docker Image'
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo '>>> Pushing Docker Image to Docker Hub'
                sh 'docker push $IMAGE_NAME'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo '>>> Deploying on Remote EC2 using Docker Compose'
                sshagent(['13.201.95.248']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} '
                            cd ${REMOTE_APP_DIR} &&
                            docker rm -f python-flask-app || true &&
                            docker-compose pull &&
                            docker-compose up -d --remove-orphans
                        '
                    """
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Build or Deployment Failed!'
        }
        success {
            echo '✅ Application Deployed Successfully!'
        }
    }
}
