pipeline {
    agent {
      label "jenkins-jx-base"
    }
    environment {
      ORG               = 'upendrasahu'
      APP_NAME          = 'phpmyadmin-9341'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    }
    stages {
      stage('Build Release') {
        when {
          branch 'master'
        }
        steps {
          container('jx-base') {
            // ensure we're not on a detached head
            sh "git checkout master"
            sh "git config --global credential.helper store"
            sh "jx step validate --min-jx-version 1.1.73"
            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
          }
          dir ('./charts/phpmyadmin-9341') {
            container('jx-base') {
              sh "make tag"
            }
          }
        }
      }
      stage('Promote to Environments') {
        when {
          branch 'master'
        }
        steps {
          dir ('./charts/phpmyadmin-9341') {
            container('jx-base') {
              // release the helm chart
              sh 'jx step helm release'

              // promote through all 'Auto' promotion Environments
              // sh 'jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)'
              sh 'jx step helm apply --namespace=jx-staging --name=phpmyadmin-9341'
            }
          }
        }
      }
    }
    post {
        always {
            cleanWs()
        }
    }
  }
