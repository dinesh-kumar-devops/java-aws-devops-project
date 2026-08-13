pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the Java application...'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh './mvnw test'
            }
        }

        stage('Archive') {
            steps {
                echo 'Archiving JAR artifact...'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

       stage('Docker Build') {
           steps {
               echo 'Building Docker image...'
               sh 'docker build -t order-service:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying application to EC2...'

                sshagent(credentials: ['ec2-order-service']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@44.201.56.180 \
                        "sudo mkdir -p /opt/order-service && sudo chown ubuntu:ubuntu /opt/order-service"

                        scp -o StrictHostKeyChecking=no \
                        target/order-service-0.0.1-SNAPSHOT.jar \
                        ubuntu@44.201.56.180:/tmp/order-service.jar

                        ssh -o StrictHostKeyChecking=no ubuntu@44.201.56.180 \
                        "sudo cp /tmp/order-service.jar /opt/order-service/order-service.jar && \
                         sudo systemctl restart order-service && \
                         sleep 5 && \
                         sudo systemctl is-active --quiet order-service"

                        echo "Application deployed successfully."
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking application health...'

                sshagent(credentials: ['ec2-order-service']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@44.201.56.180 \
                        "curl -f http://localhost:8080/health"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'CI/CD pipeline failed. Check the console logs.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
