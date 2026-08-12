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
                sh './mvnw test-FAILED'
            }
        }

        stage('Archive') {
            steps {
                echo 'Archiving JAR artifact...'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully.'
        }

        failure {
            echo 'CI pipeline failed. Check the console logs.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
