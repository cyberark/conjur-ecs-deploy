pipeline {
  agent { label 'executor-v2' }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '30'))
  }

  triggers {
    cron(getDailyCronString())
  }

  stages {
    stage('Lint'){
      steps {
        sh 'scripts/lint'
      }
    }
    // stage('Smoke Test'){
    //   // environment {
    //   //   STACK_NAME = "conjur-ecs-deploy-ci-${BUILD_NUMBER}"
    //   // }
    //   steps {
    //     env.STACK_NAME="conjur-ecs-deploy-ci-${BUILD_NUMBER}"
    //     sh 'scripts/smoktetest_deploy'
    //     sh 'scripts/smoketest_exercise'
    //   }
    //   post {
    //     always {
    //       sh 'scripts/smoketest_cleanup'
    //       archiveArtifacts(artifacts: 'params_smoketests.json')
    //       archiveArtifacts(artifacts: '*.log')
    //     }
    //   }
    // }
  }

    post {
    always {
      cleanupAndNotify(currentBuild.currentResult)
    }
  }
}
