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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://192.168.24.153:8081/')], contextPath: null, war: '**/*.war'
            }
        }

        stage('API Testing') {
            steps {
                script {
                    def timeout = 300 // Maximum timeout for waiting Tomcat deployment (in seconds)
                    def startTime = currentTimeMillis()

                    // Wait until Tomcat application is accessible
                    def tomcatAccessible = false
                    while (!tomcatAccessible && (currentTimeMillis() - startTime) < (timeout * 1000)) {
                        try {
                            sh 'curl -sSf http://192.168.24.153:8081/webapp-0.2/'
                            tomcatAccessible = true
                        } catch (Exception e) {
                            sleep time: 10, unit: 'SECONDS'
                        }
                    }
                    if (!tomcatAccessible) {
                        error 'Tomcat deployment timeout'
                    }

                    // Perform API testing and assertions
                    def getResponse = sh(script: 'curl -X GET http://192.168.138.114:8081/webapp-0.2/', returnStdout: true).trim()
                    def postResponse = sh(script: 'curl -X POST -d "n1=5&n2=6&r1=add" http://192.168.138.114:8081/webapp-0.2/firstHomePage', returnStdout: true).trim()

                    // Assert responses
                    assert getResponse.contains("ExpectedText")
                    assert postResponse.contains("ExpectedText")

                    // Print responses
                    echo "GET Response: ${getResponse}"
                    echo "POST Response: ${postResponse}"
                }
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
