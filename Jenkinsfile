@Library('shared-lib') _

pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhubpwd')
        SLACK_CHANNEL = '#build-status'
        SLACK_COLOR = '#FF0000'
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
        stage('Verify Dockerfile') {
            parallel {
                stage('Verify Java Dockerfile') {
                    steps {
                        script {
                            dir('testhello') {
                                sh 'ls -la'
                                if (!fileExists('Dockerfile')) {
                                    error 'Dockerfile not found in testhello directory'
                                }
                            }
                        }
                    }
                }
                stage('Verify Python Dockerfile') {
                    steps {
                        script {
                            dir('python-app') {
                                sh 'ls -la'
                                if (!fileExists('Dockerfile')) {
                                    error 'Dockerfile not found in python-app directory'
                                }
                            }
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
                            dir('testhello') {
                                withDockerRegistry([url: '', credentialsId: 'dockerhubpwd']) {
                                    sh 'docker build -t pramila188/testhello .'
                                    sh 'docker tag pramila188/testhello:latest index.docker.io/pramila188/testhello:latest'
                                    sh 'docker push index.docker.io/pramila188/testhello:latest'
                                }
                            }
                        }
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
            script {
                def buildStatus = currentBuild.result ?: 'SUCCESS'
                def color = buildStatus == 'SUCCESS' ? 'good' : SLACK_COLOR
                slackSend(channel: SLACK_CHANNEL, color: color, message: "${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }
    }
}
