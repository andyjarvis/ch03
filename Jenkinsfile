pipeline {
  agent none
  stages {
    stage('Build & Test') {
      agent {
        node {
          label 'docker'
        }

      }
      steps {
        sh 'mvn -Dmaven.test.failure.ignore clean package'
        stash(name: 'build-test-artifacts', includes: '**/target/surefire-reports/TEST-*.xml,target/*.jar')
      }
    }
    stage('Report & Publish') {
      parallel {
        stage('Report & Publish') {
          agent {
            node {
              label 'docker'
            }

          }
          steps {
            unstash 'build-test-artifacts'
            junit '**/target/surefire-reports/TEST-*.xml'
            archiveArtifacts(artifacts: 'target/*.jar', onlyIfSuccessful: true)
          }
        }
        stage('Publish to Artifactory') {
          agent {
            node {
              label 'docker'
            }

          }
          steps {
            script {
              unstash 'build-test-artifacts'
              def server = Artifactory.server 'artifactory'
              def uploadSpec = """{
                "files": [
                  {
                    "pattern": "target/*.jar",
                    "target": "example-repo-local/${BRANCH_NAME}/${BUILD_NUMBER}/"
                  }
                ]
              }"""
              server.upload(uploadSpec)
            }

          }
        }
      }
    }
  }
}