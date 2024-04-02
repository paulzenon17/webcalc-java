pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sriniketh03/webcalc-java.git'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean test install'
            }
        }

        

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://192.168.24.153:8081/')], contextPath: null, war: '**/*.war'
            }
        }

        
    }

    post {
        success {
            echo 'Deployment successful!'
            // Notify success, send emails, etc.
        }
        failure {
            echo 'Deployment failed!'
            // Notify failure, send emails, etc.
        }
    }
}
