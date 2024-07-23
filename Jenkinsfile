@Library('shared_library') _
import org.foo.Utilities

pipeline {
    agent any
    stages {
        stage('Build Java App') {
            steps {
                script {
                    buildJavaApp()
                }
            }
        }
        stage('Build Python App') {
            steps {
                script {
                    buildPythonApp()
                }
            }
        }
    }
    post {
        success {
            script {
                commonFunctions.sendSlackNotification("Build successful!")
            }
        }
        failure {
            script {
                commonFunctions.sendSlackNotification("Build failed!")
            }
        }
    }
}
