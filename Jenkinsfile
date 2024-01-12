pipeline {
  agent { label 'conjur-enterprise-common-agent' }

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
    stage('Get InfraPool Agents') {
      steps{
        script {
          INFRAPOOL_EXECUTORV2_AGENT_0 = getInfraPoolAgent.connected(type: "ExecutorV2", quantity: 1, duration: 1)[0]
        }
      }
    }
    stage('Lint'){
      steps {
        script {
          INFRAPOOL_EXECUTORV2_AGENT_0.agentSh 'scripts/lint'
        }
      }
    }
    stage('Validate Changelog') {
      steps {
        script {
          INFRAPOOL_EXECUTORV2_AGENT_0.agentSh 'ci/parse-changelog'
        }
      }
    }
    stage('Smoke Test'){
      environment {
        INFRAPOOL_STACK_NAME = "ecsdeployci${BRANCH_NAME.replaceAll("[^A-Za-z0-9]", "").toLowerCase().take(6)}${BUILD_NUMBER}"
      }
      steps {
        script {
          INFRAPOOL_EXECUTORV2_AGENT_0.agentSh 'summon -f scripts/secrets.yml scripts/prepare'
          INFRAPOOL_EXECUTORV2_AGENT_0.agentSh 'summon -f scripts/secrets.yml scripts/deploy'
          INFRAPOOL_EXECUTORV2_AGENT_0.agentSh 'summon -f scripts/secrets.yml scripts/exercise'
        }
      }
      post {
        always {
          script {
            INFRAPOOL_EXECUTORV2_AGENT_0.agentArchiveArtifacts(artifacts: 'params.json')
            INFRAPOOL_EXECUTORV2_AGENT_0.agentArchiveArtifacts(artifacts: 'admin_password_meta.json', allowEmptyArchive: true)
            INFRAPOOL_EXECUTORV2_AGENT_0.agentArchiveArtifacts(artifacts: 'stack_*.json')
            INFRAPOOL_EXECUTORV2_AGENT_0.agentArchiveArtifacts(artifacts: '**/*.log', allowEmptyArchive: true)
            INFRAPOOL_EXECUTORV2_AGENT_0.agentArchiveArtifacts(artifacts: 'conjur_git_commit', allowEmptyArchive: true)
            INFRAPOOL_EXECUTORV2_AGENT_0.agentSh 'summon -f scripts/secrets.yml scripts/cleanup'
          }
        }
      }
    }
  }

    post {
    always {
      releaseInfraPoolAgent(".infrapool/release_agents")
    }
  }
}
