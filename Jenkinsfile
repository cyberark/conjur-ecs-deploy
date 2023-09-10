pipeline {
  agent { label 'executor-v2' }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '30'))
  }

  parameters {
    string(name: 'CONJUR_IMAGE', defaultValue: 'cyberark/conjur:edge', description: 'Conjur image to deploy')
  }

  triggers {
    cron(getDailyCronString())
  }

  stages {
    stage('Init'){
      steps {
        sh 'echo "Init Start"'
      }
    }
    
    stage('Lint'){
      steps {
        sh 'scripts/lint'
      }
    }
    stage('Validate Changelog') {
      steps {
        sh 'ci/parse-changelog'
      }
    }
    stage('Smoke Test'){
      environment {
        STACK_NAME = "ecsdeployci${BRANCH_NAME.replaceAll("[^A-Za-z0-9]", "").toLowerCase().take(6)}${BUILD_NUMBER}"
      }
      steps {
        sh 'summon -f scripts/secrets.yml scripts/prepare'
        sh 'summon -f scripts/secrets.yml scripts/deploy'
        sh 'summon -f scripts/secrets.yml scripts/exercise'
      }
      post {
        always {
          archiveArtifacts(artifacts: 'params.json')
          archiveArtifacts(artifacts: 'admin_password_meta.json', allowEmptyArchive: true)
          archiveArtifacts(artifacts: 'stack_*.json')
          archiveArtifacts(artifacts: '**/*.log', allowEmptyArchive: true)
          archiveArtifacts(artifacts: 'conjur_git_commit', allowEmptyArchive: true)
          sh 'summon -f scripts/secrets.yml scripts/cleanup'
        }
      }
    }
  }

    post {
    always {
      cleanupAndNotify(currentBuild.currentResult)
    }
  }
}
