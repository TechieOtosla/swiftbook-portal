
pipeline {
    agent {
        label 'maven'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Publish') {
            steps {
                sh 'mvn -s /home/jenkins/.m2/settings.xml clean deploy'
            }
        }
    }

    post {
        success {
            echo 'SwiftBook build and Artifactory publishing successful!'
        }
        failure {
            echo 'SwiftBook pipeline failed. Check the console output.'
        }
    }
}
