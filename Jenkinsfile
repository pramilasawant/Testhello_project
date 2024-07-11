@Library('shared-lib') _

pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhubpwd')
        SLACK_CREDENTIALS = credentials('slackpwd')
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
                        dir('java-app') {
                            git 'https://github.com/pramilasawant/Testhello_project.git'
                        }
                    }
                }
                stage('Checkout Python Application') {
                    steps {
                        dir('python-app') {
                            git 'https://github.com/pramilasawant/phython-application.git'
                        }
                    }
                }
            }
        }
        stage('Build and Deploy') {
            parallel {
                stage('Build and Push Java Application') {
                    steps {
                        script {
                            dir('Desktop/testhello') {
                                if (fileExists('Dockerfile')) {
                                    withDockerRegistry([ credentialsId: 'dockerhubpwd', url: '' ]) {
                                        sh 'docker build -t pramila188/testhello:latest .'
                                        sh 'docker push pramila188/testhello:latest'
                                    }
                                } else {
                                    error "Dockerfile for Java Application not found"
                                }
                            }
                        }
                    }
                }
                stage('Build and Push Python Application') {
                    steps {
                        script {
                            dir('python-app') {
                                if (fileExists('Dockerfile')) {
                                    withDockerRegistry([ credentialsId: 'dockerhubpwd', url: '' ]) {
                                        sh 'docker build -t pramila188/python-app:latest .'
                                        sh 'docker push pramila188/python-app:latest'
                                    }
                                } else {
                                    error "Dockerfile for Python Application not found"
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
        }
        failure {
            script {
                slackSend(
                    channel: '#your-channel-name',
                    color: '#FF0000',
                    tokenCredentialId: 'slackpwd',
                    message: "Pipeline failed: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
                )
            }
        }
        success {
            script {
                slackSend(
                    channel: '#your-channel-name',
                    color: '#00FF00',
                    tokenCredentialId: 'slackpwd',
                    message: "Pipeline succeeded: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
                )
            }
        }
    }
}
