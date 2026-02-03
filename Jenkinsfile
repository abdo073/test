pipeline {
  agent any

  tools {
    nodejs 'node18'
  }

  stages {

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        dir('frontend') {
          sh 'npm ci'
        }
      }
    }

    stage('Build') {
      steps {
        dir('frontend') {
          sh 'npm run build'
        }
      }
    }

    stage('Test & Coverage') {
      steps {
        dir('frontend') {
          sh 'npm test -- --coverage --watchAll=false'
        }
      }
      post {
        always {
          publishHTML(target: [
            reportDir: 'frontend/coverage',
            reportFiles: 'index.html',
            reportName: 'Coverage Report'
          ])
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        dir('frontend') {
          withSonarQubeEnv('sonarqube') {
            sh '''
              sonar-scanner \
                -Dsonar.projectKey=js-project \
                -Dsonar.sources=src \
                -Dsonar.tests=src \
                -Dsonar.test.inclusions=**/*.test.js,**/*.spec.js \
                -Dsonar.exclusions=**/node_modules/**,**/build/** \
                -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
            '''
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('OWASP Dependency Check') {
      steps {
        dir('frontend') {
          dependencyCheck additionalArguments: '--scan .',
                          odcInstallation: 'OWASP-DC'
        }
      }
      post {
        always {
          dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        }
      }
    }

    stage('Trivy Scan') {
      steps {
        dir('frontend') {
          sh 'trivy fs .'
        }
      }
    }
  }

  post {
    success {
      emailext(
        subject: "✅ Jenkins SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
          <h2>Build Success 🎉</h2>
          <p><a href="${env.BUILD_URL}">View Build</a></p>
        """,
        to: 'your@email.com'
      )
    }

    failure {
      emailext(
        subject: "❌ Jenkins FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
          <h2>Build Failed 💥</h2>
          <p><a href="${env.BUILD_URL}">Check Logs</a></p>
        """,
        to: 'your@email.com'
      )
    }
  }
}
