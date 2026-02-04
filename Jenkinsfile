pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                echo "This stage might fail"
                sh 'exit 1' // مثال فشل
            }
        }
    }

    post {
        always {
            // أي حالة: success, failure, aborted
            notifyN8N(currentBuild.currentResult, "Pipeline finished")
        }
    }
}

def notifyN8N(status, reason) {
    sh """
    curl -X POST https://<N8N-PRODUCTION-WEBHOOK>/jenkins-status \
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
