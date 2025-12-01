pipeline {
    agent any

    // Optional: only keep this if you actually configured these in Jenkins
    tools {
        maven 'maven3'   // Manage Jenkins -> Global Tool Configuration -> Maven
        jdk 'JDK17'      // Manage Jenkins -> Global Tool Configuration -> JDK
    }

    environment {
        PROJECT_KEY   = 'student-management'
        PROJECT_NAME  = 'Student Management System'
        SONARQUBE_URL = 'http://localhost:9000'
        EMAIL_TO      = 'dalhoumix@gmail.com'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "📥 Repository cloned successfully"
            }
        }

        stage('Build & Tests') {
            steps {
                echo "🔨 Building and running tests..."
                // Simple & robust: run all tests
                sh 'mvn clean test'
            }
            post {
                always {
                    // Publish JUnit reports to Jenkins
                    junit 'target/surefire-reports/*.xml'
                    echo "📊 Test reports published"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Analyzing code quality with SonarQube..."
                // 'sonarqube' must match the name you configured in Jenkins (Manage Jenkins -> Configure System -> SonarQube servers)
                withSonarQubeEnv('sonarqube') {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${PROJECT_KEY} \
                          -Dsonar.projectName="${PROJECT_NAME}" \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.sources=src/main/java \
                          -Dsonar.tests=src/test/java
                    """
                }
            }
        }

        stage('Package') {
            steps {
                echo "📦 Packaging application..."
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "🎓 Pipeline finished: ${currentBuild.currentResult}"
            echo "📈 Build URL: ${env.BUILD_URL}"
        }
        success {
            mail to: "${EMAIL_TO}",
                 subject: "✅ SUCCESS - Student Management Build #${env.BUILD_NUMBER}",
                 body: """
                 Build successful! ✅

                 - Project: ${PROJECT_NAME}
                 - Build:   #${env.BUILD_NUMBER}
                 - Result:  ${currentBuild.currentResult}

                 🔗 Build URL: ${env.BUILD_URL}
                 🔗 SonarQube: ${SONARQUBE_URL}/dashboard?id=${PROJECT_KEY}
                 """
        }
        failure {
            mail to: "${EMAIL_TO}",
                 subject: "❌ FAILED - Student Management Build #${env.BUILD_NUMBER}",
                 body: """
                 Build failed ❌

                 Check details here:
                 ${env.BUILD_URL}
                 """
        }
    }
}
