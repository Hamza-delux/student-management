pipeline {
    agent any

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
                sh 'mvn clean test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    echo "📊 Test reports published"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Analyzing code quality with SonarQube..."
                // Name here must match your SonarQube server in Jenkins config
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
            echo "🎓 Pipeline finished: ${currentBuild.currentR
