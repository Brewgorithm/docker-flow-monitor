import java.text.SimpleDateFormat

pipeline {
  agent {
    label "test"
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '2'))
    disableConcurrentBuilds()
  }
  stages {
    stage("build") {
      steps {
        script {
          def dateFormat = new SimpleDateFormat("yy.MM.dd")
          currentBuild.displayName = dateFormat.format(new Date()) + "-" + env.BUILD_NUMBER
        }
        sh "docker image build -t registry.brewgorithm.com/docker-flow-monitor ."
        // sh "docker image build -t registry.brewgorithm.com/docker-flow-monitor-docs -f Dockerfile.docs ."
      }
    }
    stage("release") {
      when {
        branch "master"
      }
      steps {
        sh "docker tag registry.brewgorithm.com/docker-flow-monitor registry.brewgorithm.com/docker-flow-monitor:${currentBuild.displayName}"
        sh "docker image push registry.brewgorithm.com/docker-flow-monitor:latest"
        sh "docker image push registry.brewgorithm.com/docker-flow-monitor:${currentBuild.displayName}"
      }
    }
  }
  post {
    always {
      sh "docker system prune -f"
    }
    failure {
      slackSend(
        color: "danger",
        message: "${env.JOB_NAME} failed: ${env.RUN_DISPLAY_URL}"
      )
    }
  }
}
