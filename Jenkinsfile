pipeline {
  agent any

  environment {
    PROJECT_KEY  = 'student-management'
    PROJECT_NAME = 'Student Management System'
    EMAIL_TO     = 'dalhoumix@gmail.com'
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
        sh 'mvn -B clean test'
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
        withSonarQubeEnv('My SonarQube Server') {
          sh """
            mvn -B sonar:sonar \
              -Dsonar.projectKey="${PROJECT_KEY}" \
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
        sh 'mvn -B package -DskipTests'
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
      mail(
        to: EMAIL_TO,
        subject: "✅ SUCCESS - Student Management Build #${env.BUILD_NUMBER}",
        body: """Build successful!
Project: ${PROJECT_NAME}
Build: #${env.BUILD_NUMBER}
Result: ${currentBuild.currentResult}
Build URL: ${env.BUILD_URL}
SonarQube project: ${PROJECT_KEY}
"""
      )
    }
    failure {
      mail(
        to: EMAIL_TO,
        subject: "❌ FAILED - Student Management Build #${env.BUILD_NUMBER}",
        body: """Build failed!
Project: ${PROJECT_NAME}
Build: #${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}
"""
      )
    }
  }
}
