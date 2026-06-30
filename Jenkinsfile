pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    sh '''
                    mvn deploy -DskipTests -s $MAVEN_SETTINGS
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-creds',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )
                ]) {

                    sshagent(['app-server']) {

                        sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@172.31.26.228 << EOF

                        mkdir -p /opt/payment-service
                        cd /opt/payment-service

                        wget \
                        --user=$NEXUS_USER \
                        --password=$NEXUS_PASS \
                        -O payment-service.jar \
                        "http://16.113.30.241:8081/repository/maven-snapshots/com/company/payment-service/0.0.1-SNAPSHOT/payment-service-0.0.1-20260630.094926-1.jar"

                        pkill -f payment-service || true

                        nohup java -jar payment-service.jar > app.log 2>&1 &

                        EOF
                        '''
                    }
                }
            }
        }
    }
}
