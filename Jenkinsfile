def appName=env.APP_NAME
def gitSourceUrl=env.GIT_SOURCE_URL
def gitSourceRef=env.GIT_SOURCE_REF


pipeline {
  agent {
    label 'node12'
  }
  stages {

    stage('Initialize') {
      steps {
        echo "appName: ${appName}"
        echo "gitSourceUrl: ${gitSourceUrl}"
        echo "gitSourceRef: ${gitSourceRef}"
      }
    }
    stage('Checkout') {
      steps {
        echo "Checkout source."
        git url: "${gitSourceUrl}", branch: "${gitSourceRef}"
      }
    }
    stage('npm run build') {
      steps {
        echo "Build the app."
        sh "npm update"
        sh "npm install -g @angular/cli"
        sh "npm run build"
      }
    }               
    stage('Echo output') {
      steps {
        script {
          echo "Did the build work?"
          echo "List current dir."
          sh "ls -la"
          echo "List dist dir."
          sh "ls -la dist"
        }
      }
    }
    stage('Build Image') {
      steps {
        script {
          echo "Build container image."
          openshift.withCluster() {
            openshift.withProject('mean') {
              sh "oc start-build mean-frontend-s2i-build --from-dir=dist/mean-contactlist-angular2 --follow"
            }
          }
        }
      }
    }
  }
}
