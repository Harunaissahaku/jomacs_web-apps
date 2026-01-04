
pipeline {
    agent any

    tools {
        maven 'maven3.9.11'
        jdk 'jdk17'   // must be configured in Jenkins → Global Tool Configuration
    }

    environment {
        MAVEN_OPTS = "-Dmaven.test.failure.ignore=true"
    }

    stages {

        stage('1. Checkout Source Code') {
            steps {
                echo 'Cloning source code...'
                git branch: 'main',
                    url: 'https://github.com/JOMACS-IT/web-app.git'
            }
        }

        stage('2. Build & Unit Test') {
            steps {
                echo 'Building application...'
                sh 'mvn clean verify'
            }
        }

        stage('3. SonarQube Code Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('4. Quality Gate (Optional but Recommended)') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('5. Publish Artifact to Nexus') {
            steps {
                echo 'Publishing artifact to Nexus...'
                sh 'mvn deploy -DskipTests'
            }
        }

        stage('6. Deploy to Tomcat (UAT)') {
            steps {
                echo 'Deploying WAR to Tomcat...'
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat_cred',
                        path: '',
                        url: 'http://18.117.162.68:9090'
                    )
                ],
                contextPath: null,
                war: 'target/*.war'
            }
        }

    }

    post {
        success {
            emailext(
                subject: '✅ Deployment Successful',
                body: '''
Hello Team,

The Jenkins pipeline completed successfully.

✔ Build
✔ SonarQube Scan
✔ Quality Gate Passed
✔ Nexus Publish
✔ Tomcat Deployment

Regards,
Jenkins CI
''',
                to: 'info@jomacsit.com'
            )
        }

        failure {
            emailext(
                subject: '❌ Deployment Failed',
                body: '''
Hello Team,

The Jenkins pipeline FAILED.

Please check Jenkins console logs for details.

Regards,
Jenkins CI
''',
                to: 'info@jomacsit.com'
            )
        }
    }
}
