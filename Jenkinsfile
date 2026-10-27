pipeline {
    options { timestamps() }
    agent none
    environment {
        DOCKER_USER = 'w1pyaa'
        IMAGE_NAME = 'lab4-flask-app'
    }
    stages {
        stage('Check scm') {
            agent any
            steps {
                checkout scm
            }
        }
        stage('Build') {
            agent any
            steps {
                echo "Building ...${BUILD_NUMBER}"
                echo "Build completed"
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'python:3.11-alpine'
                    args '-u root'
                }
            }
            steps {
                sh 'pip install --no-cache-dir Flask unittest-xml-reporting'
                sh 'python app_tests.py'
            }
            post {
                always {
                    junit testResults: 'test-reports/*.xml', allowEmptyResults: true
                }
                success {
                    echo "Application testing successfully completed"
                }
                failure {
                    echo "Oooppss!!! Tests failed!"
                }
            }
        }
        stage('Docker Build & Push') {
            agent any
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                        sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                        sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} -t ${DOCKER_USER}/${IMAGE_NAME}:latest ."
                        sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }
    }
}
