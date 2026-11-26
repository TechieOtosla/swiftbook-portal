
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

        stage('Deploy to Staging') {
            steps {
                sh '''
                    scp -o BatchMode=yes \
                        -i /home/jenkins/.ssh/id_ed25519 \
                        target/swiftbook-portal.war \
                        swiftbook-deploy@172.31.43.8:/opt/tomcat-staging/webapps/
                '''
            }
        }
    }

    post {
        success {
            echo 'SwiftBook build, publishing and staging deployment successful!'
        }
        failure {
            echo 'SwiftBook pipeline failed. Check the console output.'
        }
    }
}
