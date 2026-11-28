
pipeline {
    agent {
        label 'maven'
    }

    environment {
        EXPECTED_VERSION = '1.0.3'
        EXPECTED_SHA256 = '4105a6f92ee6e0b4efd7e1889e21fa05a03213f3a83b7821a53b072f4b3295bc'
        TOMCAT_HOST = '172.31.43.8'
        DEPLOY_USER = 'swiftbook-deploy'
        SSH_KEY = '/home/jenkins/.ssh/id_ed25519'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -eu

                    mvn -s /home/jenkins/.m2/settings.xml clean package

                    unzip -p target/swiftbook-portal.war \
                        META-INF/maven/com.swiftbook/swiftbook-portal/pom.properties \
                        | grep -Fx "version=${EXPECTED_VERSION}"

                    sha256sum target/swiftbook-portal.war \
                        | grep -F "${EXPECTED_SHA256}"

                    echo "Build and artifact verification successful."
                '''
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh '''
                    set -eu

                    scp -o BatchMode=yes \
                        -i "${SSH_KEY}" \
                        target/swiftbook-portal.war \
                        "${DEPLOY_USER}@${TOMCAT_HOST}:/opt/tomcat-staging/webapps/"

                     ssh -o BatchMode=yes \
    -i "${SSH_KEY}" \
    "${DEPLOY_USER}@${TOMCAT_HOST}" \
    'set -eu
     for attempt in $(seq 1 12); do
         if sha256sum /opt/tomcat-staging/webapps/swiftbook-portal.war | grep -F "4105a6f92ee6e0b4efd7e1889e21fa05a03213f3a83b7821a53b072f4b3295bc" \
            && grep -Fqx "version=1.0.3" /opt/tomcat-staging/webapps/swiftbook-portal/META-INF/maven/com.swiftbook/swiftbook-portal/pom.properties \
            && curl -fsS --connect-timeout 5 --max-time 15 http://127.0.0.1:8180/swiftbook-portal/ >/dev/null; then
             echo "Staging deployment verified successfully."
             exit 0
         fi
         echo "Waiting for staging deployment verification (attempt $attempt of 12)..."
         sleep 5
     done
     echo "Staging verification failed after 60 seconds."
     exit 1'
                '''
            }
        }

        stage('Approve Production') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input message: 'Approve SwiftBook 1.0.3 production deployment? Update the production deployment script version and checksum pin before proceeding.', ok: 'Proceed with production'
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                sh '''
                    set -eu

                    scp -o BatchMode=yes \
                        -i "${SSH_KEY}" \
                        target/swiftbook-portal.war \
                        "${DEPLOY_USER}@${TOMCAT_HOST}:/opt/swiftbook-incoming/swiftbook-portal.war"

                    ssh -o BatchMode=yes \
                        -i "${SSH_KEY}" \
                        "${DEPLOY_USER}@${TOMCAT_HOST}" \
                        'set -eu
                         sudo -n /usr/local/sbin/deploy-swiftbook-production.sh'
                '''
            }
        }

        stage('Verify Production') {
            steps {
                sh '''
                    set -eu

                    ssh -o BatchMode=yes \
                        -i "${SSH_KEY}" \
                        "${DEPLOY_USER}@${TOMCAT_HOST}" \
                        'set -eu
                         attempt=1
                         while [ "$attempt" -le 12 ]; do
                             if sha256sum /opt/tomcat/webapps/swiftbook-portal.war | grep -F "4105a6f92ee6e0b4efd7e1889e21fa05a03213f3a83b7821a53b072f4b3295bc" \
                                && grep -Fqx "version=1.0.3" /opt/tomcat/webapps/swiftbook-portal/META-INF/maven/com.swiftbook/swiftbook-portal/pom.properties \
                                && curl -fsS --connect-timeout 5 --max-time 15 http://127.0.0.1:8080/swiftbook-portal/ >/dev/null; then
                                 echo "Production deployment verified successfully."
                                 exit 0
                             fi
                             echo "Waiting for production verification (attempt $attempt of 12)..."
                             sleep 5
                             attempt=$((attempt + 1))
                         done
                         echo "Production verification failed after 60 seconds."
                         exit 1'
                '''
            }
        }
    }

    post {
        success {
            echo 'SwiftBook build, staging deployment and production verification successful!'
        }

        failure {
            echo 'SwiftBook pipeline failed. Check the Jenkins console output.'
        }

        aborted {
            echo 'SwiftBook pipeline was aborted.'
        }
    }
}
