#!/usr/bin/env groovy

pipeline {
  agent { label 'executor-v2' }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '30'))
  }

  stages {
    stage('Validate') {
      parallel {
        stage('Changelog') {
          steps { sh './bin/parse-changelog.sh' }
        }
      }
    }

    stage('Build client Docker image') {
      steps {
        sh './bin/build'
      }
    }

    stage('Run Tests') {
      steps {
        sh './bin/test'

        junit 'test/junit.xml'
      }
    }

    stage("Scan images") {
      parallel {
        stage ("Scan main image for fixable vulns") {
          steps {
            scanAndReport("conjur-authn-k8s-client:dev", "HIGH", false)
          }
        }
        stage ("Scan main image for total vulns") {
          steps {
            scanAndReport("conjur-authn-k8s-client:dev", "NONE", true)
          }
        }
        stage ("Scan redhat image for fixable vulns") {
          steps {
            scanAndReport("conjur-authn-k8s-client-redhat:dev", "HIGH", false)
          }
        }
        stage ("Scan redhat image for total vulns") {
          steps {
            scanAndReport("conjur-authn-k8s-client-redhat:dev", "NONE", true)
          }
        }
      }
    }

    stage('Publish client Docker image') {
      when {
        branch 'master'
      }
      steps {
        sh 'summon ./bin/publish'
      }
    }
  }

  post {
    always {
      cleanupAndNotify(currentBuild.currentResult)
    }
  }
}
