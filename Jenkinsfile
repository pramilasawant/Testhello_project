@Library('shared-lib') _

pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhubpwd')
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                deleteDir()
            }
        }
        stage('Checkout Repositories') {
            parallel {
                stage('Checkout Java Application') {
                    steps {
                        dir('testhello') {
                            git 'https://github.com/pramilasawant/Testhello_project.git'
                        }
                    }
                }
                stage('Checkout Python Application') {
                    steps {
                        dir('python-app') {
                             git branch: 'main', url: 'https://github.com/pramilasawant/phython-application.git'
                        }
                    }
                }
            }
        }
        stage('Build and Deploy') {
            parallel {
                stage('Build and Push Java Application') {
                  steps {
                    sh 'ls'
                    sh 'docker build -t pramila188/testhello .'
        }
      }


                stage('Build and Push Python Application') {
                    steps {
                        script {
                            dir('python-app') {
                                withDockerRegistry([url: '', credentialsId: 'dockerhubpwd']) {
                                    sh 'docker build -t pramila188/python-app:latest .'
                                    sh 'docker tag pramila188/python-app:latest index.docker.io/pramila188/python-app:latest'
                                    sh 'docker push index.docker.io/pramila188/python-app:latest'
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
            slackSend(channel: '#build-status', color: '#FF0000', message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}
