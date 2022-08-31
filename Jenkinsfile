pipeline {
  agent any
  stages {
    stage('Deploy start') {
      steps {
        slackSend(message: "Deploy ${env.BUILD_NUMBER} Started ${env.BUILD_URL}"
        , color: 'good', tokenCredentialId: 'slack-key')
      }
    }      
    stage('git pull') {
      steps {
        slackSend(message: "Deploy ${env.BUILD_NUMBER} git pull Started"
        , color: 'good', tokenCredentialId: 'slack-key')
        // https://github.com/kuuku123/GitOps.git will replace by sed command before RUN
        git url: 'https://github.com/kuuku123/GitOps.git', branch: 'main'
      }
    }
    stage('k8s deploy'){
      steps {
        slackSend(message: "Deploy ${env.BUILD_NUMBER} k8s deploy Started"
        , color: 'good', tokenCredentialId: 'slack-key', failOnError: true)
        kubernetesDeploy(kubeconfigId: 'kubeconfig',
                         configs: '*.yaml')
      }
    }
    stage('send diff') {
      steps {
        script {
          def publisher = LastChanges.getLastChangesPublisher "PREVIOUS_REVISION", "SIDE", "LINE", true, true, "", "", "", "", ""
          publisher.publishLastChanges()
          def htmlDiff = publisher.getHtmlDiff()
          writeFile file: "deploy-diff-${env.BUILD_NUMBER}.html", text: htmlDiff
        }
        slackSend(message: """${env.JOB_NAME} #${env.BUILD_NUMBER} End
        (<${env.BUILD_URL}/last-changes|Check Last changed>)"""
        , color: 'good', tokenCredentialId: 'slack-key')             
      }
    }
  }
  post {
      success {
          slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})",tokenCredentialId: 'slack-key')
      }
      failure {
          slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})",tokenCredentialId: 'slack-key')
      }
  }
}
