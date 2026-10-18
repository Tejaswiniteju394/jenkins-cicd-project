pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        IMAGE_NAME = 'jenkins-cicd-website'
        CONTAINER_NAME = 'jenkins-cicd-website'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker image...'

                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing the website container...'

                sh '''
                    set -eu
                    docker rm -f jenkins-test || true

                    cleanup() {
                        docker rm -f jenkins-test >/dev/null 2>&1 || true
                    }

                    trap cleanup EXIT

                    docker run -d --name jenkins-test \
                        -p 127.0.0.1:8081:80 \
                        "$IMAGE_NAME:$BUILD_NUMBER"

                    for i in $(seq 1 10); do
                        if curl -fsS http://127.0.0.1:8081/ \
                            -o /tmp/jenkins-test.html; then
                            break
                        fi
                        sleep 2
                    done

                    grep -q 'Hello from AWS EC2!' /tmp/jenkins-test.html
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying website on AWS EC2...'

                sh '''
                    docker rm -f "$CONTAINER_NAME" || true

                    docker run -d \
                        --restart unless-stopped \
                        --name "$CONTAINER_NAME" \
                        -p 80:80 \
                        "$IMAGE_NAME:$BUILD_NUMBER"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
            echo 'Website is deployed on AWS EC2 port 80.'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
