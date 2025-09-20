pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  parameters {
    string(name: 'EMAIL_TO', defaultValue: 'your.email@example.com',
           description: 'Recipients for stage notifications')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Check Tools') {
      steps {
        bat 'cmd /c node -v'
        bat 'cmd /c npm -v'
        bat 'cmd /c where node'
        bat 'cmd /c where npm'
      }
    }

    stage('Install Dependencies') {
      steps { bat 'cmd /c npm install' }
    }

    stage('Run Tests') {
      steps {
        script {
          int code = bat(returnStatus: true, script: 'cmd /c npm test > test.log 2>&1')
          bat 'cmd /c type test.log'
          if (code != 0) { currentBuild.result = 'UNSTABLE' }
        }
      }
      post {
        always {
          emailext(
            to: params.EMAIL_TO,
            subject: "[${currentBuild.result}] ${env.JOB_NAME} #${env.BUILD_NUMBER} — Run Tests",
            mimeType: 'text/html',
            body: """
              <h3>Stage: Run Tests</h3>
              <p>Status: <b>${currentBuild.result}</b></p>
              <p>Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}</p>
              <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
            """,
            attachmentsPattern: 'test.log',
            attachLog: true,
            compressLog: true
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        bat 'cmd /c npm run coverage || exit /b 0'
        bat 'cmd /c if exist coverage\\lcov.info (echo coverage present) else (echo no coverage)'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        script {
          int code = bat(returnStatus: true, script: 'cmd /c npm audit --json > audit.json 2>&1')
          bat 'cmd /c type audit.json'
          if (code != 0) { currentBuild.result = 'UNSTABLE' }
        }
      }
      post {
        always {
          emailext(
            to: params.EMAIL_TO,
            subject: "[${currentBuild.result}] ${env.JOB_NAME} #${env.BUILD_NUMBER} — Security Scan",
            mimeType: 'text/html',
            body: """
              <h3>Stage: NPM Audit (Security Scan)</h3>
              <p>Status: <b>${currentBuild.result}</b></p>
              <p>Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}</p>
              <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
              <p>Attached: <code>audit.json</code> and the full console log.</p>
            """,
            attachmentsPattern: 'audit.json',
            attachLog: true,
            compressLog: true
          )
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'test.log,audit.json,coverage/**', allowEmptyArchive: true
    }
  }
}
