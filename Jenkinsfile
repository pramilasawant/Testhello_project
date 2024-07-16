pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhunpwd')
        SLACK_CREDENTIAL_ID = 'slackpwd'
        JAVA_APP_REPO = 'https://github.com/pramilasawant/Testhello_project.git'
        PYTHON_APP_REPO = 'https://github.com/pramilasawant/phython-application.git'
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
                            git url: "${JAVA_APP_REPO}", branch: 'main'
                        }
                    }
                }
                stage('Checkout Python Application') {
                    steps {
                        dir('python-app') {
                            git url: "${PYTHON_APP_REPO}", branch: 'main'
                        }
                    }
                }
            }
        }

        stage('Build and Deploy') {
            parallel {
                stage('Build and Push Java Application') {
                    steps {
                        dir('testhello') {
                            script {
                                docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                                    sh 'docker build -t pramila188/testhello .'
                                    sh 'docker push pramila188/testhello'
                                }
                            }
                        }
                    }
                }
                stage('Build and Push Python Application') {
                    steps {
                        dir('python-app') {
                            script {
                                docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                                    sh 'docker build -t pramila188/python-app:latest .'
                                    sh 'docker push pramila188/python-app:latest'
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
                try {
                    slackSend(channel: '#build-status', color: '#FF0000', message: "Build ${currentBuild.fullDisplayName} failed", tokenCredentialId: SLACK_CREDENTIAL_ID)
                } catch (e) {
                    echo "Slack notification failed: ${e.message}"
                }
            }
        }
        success {
            script {
                try {
                    slackSend(channel: '#build-status', color: '#00FF00', message: "Build ${currentBuild.fullDisplayName} succeeded", tokenCredentialId: SLACK_CREDENTIAL_ID)
                } catch (e) {
                    echo "Slack notification failed: ${e.message}"
                }
            }
        }
    }
}
