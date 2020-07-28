def appName=env.APP_NAME
def gitSourceUrl=env.GIT_SOURCE_URL
def gitSourceRef=env.GIT_SOURCE_REF
def projectBase=env.PROJECT_BASE

pipeline {
  agent {
    label 'node12'
  }
  stages {

    stage('Initialize') {
      steps {
        echo "appName: ${appName}"
        echo "projectBase: ${projectBase}"
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
    stage('Build Image') {
      steps {
        script {
          echo "Build container image."
          openshift.withCluster() {
            openshift.withProject('cicd') {
              sh "oc start-build ${appName}-s2i-build --from-dir=dist/mean-contactlist-angular2 --follow"
            }
          }
        }
      }
    }
    stage("Tag DEV") {
      steps {
        script {
          echo "Tag image to DEV"
          openshift.withCluster() {
            openshift.withProject('cicd') {
              openshift.tag("${appName}:latest", "${appName}:dev")
            }
          }
        }
      }
    }
    stage('Deploy DEV') {
      steps {
        script {
          echo "Deploy to DEV."
          openshift.withCluster() {
            openshift.withProject("${projectBase}-dev") {
              echo "Rolling out ${appName} to DEV."
              def dc = openshift.selector('dc', "${appName}")
              dc.rollout().latest()
              dc.rollout().status()
            }
          }
        }
      }
    }
    stage('Integration Tests') {
      steps {
        echo "Running Integration tests..."
        //sh "mvn verify -Pfailsafe"
      }
    }
    stage("Tag UAT") {
      steps {
        script {
          echo "Tag ${appName} image to UAT"
          openshift.withCluster() {
            openshift.withProject('cicd') {
              openshift.tag("${appName}:dev", "${appName}:uat")
            }
          }
        }
      }
    }
    stage('Deploy UAT') {
      steps {
        script {
          echo "Deploy to UAT."
          openshift.withCluster() {
            openshift.withProject("${projectBase}-uat") {
              echo "Rolling out ${appName} to UAT."
              def dc = openshift.selector('dc', "${appName}")
              dc.rollout().latest()
              dc.rollout().status()
            }
          }
        }
      }
    }
  }
}
