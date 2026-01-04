


pipeline {
    agent any

    tools {
        maven "maven3.9.11" // Make sure this is configured in Jenkins
        jdk "jdk17"          // Make sure JDK17 is installed in Jenkins tools
    }

    environment {
        SONAR_HOST_URL = "http://localhost:9000"           // Change to your SonarQube URL
        SONAR_TOKEN = credentials('sonar-token-id')       // Jenkins credential ID for Sonar token
    }

    stages {
        stage("1. Git clone from repo") {
            steps {
                echo "Start Git clone"
                git branch: 'main', url: 'https://github.com/JOMACS-IT/web-app.git'
                echo "End Git clone"
            }
        }

        stage("2. Build with Maven") {
            steps {
                echo "Start Maven build"
                sh "mvn clean package"
                echo "End Maven build"
            }
        }

        stage("3. SonarQube Code Scan") {
            steps {
                echo "Start SonarQube scan"
                sh """
                mvn sonar:sonar \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_TOKEN}
                """
                echo "End SonarQube scan"
            }
        }

        stage("4. Store Artifacts") {
            steps {
                echo "Deploy Artifact to Nexus (or other repo)"
                sh "mvn deploy"
            }
        }

        stage("5. Deploy to Tomcat UAT") {
            steps {
                echo "Deploying to Tomcat in UAT"
                deploy adapters: [tomcat9(
                    credentialsId: 'tomcat_cred',
                    path: '',
                    url: 'http://18.117.162.68:9090'
                )], contextPath: null, war: 'target/*.war'
            }
        }

        stage("6. Email Notification") {
            steps {
                echo "Sending Email Notification"
                emailext body: 'The Deployment is Successful', 
                         subject: 'Deployment Success', 
                         to: 'info@jomacsit.com'
            }
        }
    }
}
