pipeline {
  agent any
  stages {
    stage('Parallel steps') {
      parallel {
        stage('Parallel 1') {
          steps {
            echo 'Parallel Step 1'
          }
        }
        stage('Parallel 2') {
          steps {
            pwd()
          }
        }
        stage('Parallel 3') {
          steps {
            slackSend(message: 'Send from Parallel Step 3', color: 'good')
          }
        }
      }
    }
    stage('Simple stage') {
      steps {
        retry(count: 6) {
          echo 'Could be print at least 6 times'
        }

      }
    }
    stage('FIltered stage') {
      steps {
        sh 'curl http://www.google.com'
      }
    }
  }
}