pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub'
        DOCKERHUB_REPO = 'pramila188'
        SLACK_CHANNEL = '#build-notifications'
        SLACK_CREDENTIAL_ID = 'slackpwd'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Checkout Java Application') {
            steps {
                dir('java-app') {
                    git url: 'https://github.com/pramilasawant/Testhello_project.git', branch: 'main'
                }
            }
        }

        stage('Build Java Application') {
            steps {
                dir('var/lib/jenkins/workspace/j-p-project/java-app/Hello/testhello') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Java Docker Image') {
            steps {
                dir('java-app') {
                    script {
                        dockerImage = docker.build("${DOCKERHUB_REPO}/java-app:${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Push Java Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy Java Application') {
            steps {
                script {
                    kubectlDeploy([
                        kubeconfigId: 'kubeconfig', 
                        configs: 'java-app/k8s/deployment.yaml', 
                        enableConfigSubstitution: true
                    ])
                }
            }
        }

        stage('Checkout Python Application') {
            steps {
                dir('python-app') {
                    git url: 'https://github.com/pramilasawant/phython-application.git', branch: 'main'
                }
            }
        }

        stage('Build Python Application') {
            steps {
                dir('python-app') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Build Python Docker Image') {
            steps {
                dir('python-app') {
                    script {
                        dockerImage = docker.build("${DOCKERHUB_REPO}/python-app:${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Push Python Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy Python Application') {
            steps {
                script {
                    kubectlDeploy([
                        kubeconfigId: 'kubeconfig', 
                        configs: 'python-app/k8s/deployment.yaml', 
                        enableConfigSubstitution: true
                    ])
                }
            }
        }
    }

    post {
        success {
            slackSend channel: SLACK_CHANNEL, color: 'good', message: "Build ${env.BUILD_NUMBER} succeeded.", tokenCredentialId: SLACK_CREDENTIAL_ID
        }
        failure {
            slackSend channel: SLACK_CHANNEL, color: 'danger', message: "Build ${env.BUILD_NUMBER} failed.", tokenCredentialId: SLACK_CREDENTIAL_ID
        }
    }
}
