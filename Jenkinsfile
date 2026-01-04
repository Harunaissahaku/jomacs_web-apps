pipeline {
    agent any

    tools {
        maven "maven3.9.11"  // Make sure this matches your Jenkins Maven installation
        // Optionally add JDK if needed
        // jdk "jdk17"
    }

    environment {
        SONAR_HOST_URL = 'http://localhost:9000' // Your SonarQube server
    }

    stages {

        stage("1. Git clone from repo") {
            steps {
                echo "Start Git clone"
                git branch: 'main', url: 'https://github.com/JOMACS-IT/web-app.git'
                echo "End Git clone"
            }
        }

        stage("2. Build from Maven") {
            steps {
                echo "Start building from Maven"
                sh "mvn clean package"
                echo "End build"
            }
        }

        stage("3. SonarQube Code Scan") {
            steps {
                echo "Start SonarQube scan"
                // Use Jenkins credentials securely
                withCredentials([string(credentialsId: 'sonar-token-id', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN'
                }
                echo "End SonarQube scan"
            }
        }

        stage("4. Store Artifacts") {
            steps {
                echo "Deploy Artifact"
                sh "mvn deploy"
            }
        }

        stage("5. Deploy to Tomcat in UAT") {
            steps {
                echo "Deploying to UAT server"
                deploy adapters: [tomcat9(credentialsId: 'tomcat_cred', path: '', url: 'http://18.117.162.68:9090')],
                       contextPath: null,
                       war: 'target/*.war'
            }
        }

        stage("6. Email Notification") {
            steps {
                echo "Send Email Notification"
                emailext body: 'The Deployment is Successful',
                         subject: 'Deployment Success',
                         to: 'info@jomacsit.com'
            }
        }
    }
}
