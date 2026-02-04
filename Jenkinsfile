pipeline {
  agent any

  tools {
    nodejs 'node18'
  }

  environment {
    N8N_WEBHOOK_URL = "https://abdo073.app.n8n.cloud/webhook-test/jenkins-status"
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
          sh 'npm install'
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
          archiveArtifacts artifacts: 'frontend/coverage/**', fingerprint: true
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
      notifyN8N("SUCCESS", "All stages passed successfully")
    }
    failure {
      notifyN8N("FAILURE", currentBuild.currentResult)
    }
    aborted {
      notifyN8N("ABORTED", "Pipeline was aborted (Quality Gate / timeout)")
    }
  }
}

def notifyN8N(status, reason) {
  sh """
    curl -X POST ${env.N8N_WEBHOOK_URL} \
    -H "Content-Type: application/json" \
    -d '{
      "job_name": "${env.JOB_NAME}",
      "build_number": "${env.BUILD_NUMBER}",
      "status": "${status}",
      "reason": "${reason}",
      "build_url": "${env.BUILD_URL}"
    }'
  """
}
