@Library('shared-lib') _

pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhubpwd')
        DOCKERHUB_USERNAME = 'pramila188'
        SLACK_CREDENTIALS = 'slackpwd'
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                deleteDir() // Deletes all files in the workspace
            }
        }

        stage('Checkout Repositories') {
            parallel {
                stage('Checkout Java Application') {
                    steps {
                        dir('java-app') {
                            git branch: 'main', url: 'https://github.com/pramilasawant/Testhello_project.git'
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
                        script {
                              git url: 'https://github.com/pramilasawant/Testhello_project.git', branch: 'main'

                            dir('Hello/testhello') {
                                sh 'mvn clean package'
                                docker.withRegistry('https://index.docker.io/v1/', 'dockerhubpwd') {
                                    def javaImage = docker.build("${DOCKERHUB_USERNAME}/testhello:latest", '.')
                                    javaImage.push()
                                    java_build(imageName: 'testhello')
                                  
                                }
                            }
                        }
                    }
                }

                stage('Build and Push Python Application') {
                    steps {
                        script {
                            git url: 'https://github.com/pramilasawant/phython-application.git' branch: 'main'
                            
                            dir('python-app') {
                                docker.withRegistry('https://index.docker.io/v1/', 'dockerhubpwd') {
                                    def pythonImage = docker.build("${DOCKERHUB_USERNAME}/python-app:latest", '.')
                                    pythonImage.push()
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
            cleanWs() // Clean workspace after build
        }
        success {
            slackSend (color: '#00FF00', message: "Build succeeded: ${env.JOB_NAME} ${env.BUILD_NUMBER}", tokenCredentialId: SLACK_CREDENTIALS)
        }
        failure {
            slackSend (color: '#FF0000', message: "Build failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}", tokenCredentialId: SLACK_CREDENTIALS)
        }
    }
}
