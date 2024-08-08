pipeline {
  agent any

  tools {
      maven 'MavenTool'
      jdk 'OracleJDK8'
  }

  environment {
    SSH_CREDENTIALS_ID = 'ssh_to_gitlab'
    GIT_URL = '<project_path>.git'
    REGISTRY_USERNAME = 'jenkins@gmail.com'
    SETTINGS_XML_FILE_ID = 'artifactory_settings_xml'
    CREDENTIALS_ID = 'artifactory_admin_credentials'
  }

  stages {
    stage('Initialize') {
      steps {
        deleteDir()
        checkout scm
      }
    }

    stage('Calculate Tag') {
      when {
        anyOf {
          branch 'release/*'
        }
      }
      
      steps {
        sshagent(['ssh_to_gitlab']) {
          script {
            def version = GIT_BRANCH.split('/')[-1]
            MAJOR_VERSION = version.split('\\.')[0]
            MINOR_VERSION = version.split('\\.')[1]

            sh 'git fetch --tags'
            def latestPatch = sh( script: "git tag -l ${MAJOR_VERSION}.${MINOR_VERSION}.* | sort -V | tail -n1 | cut -d '.' -f 3", returnStdout: true).trim()
            if (latestPatch.length() != 0) {
              PATCH_NUMBER = latestPatch.toInteger() + 1
            } else {
              PATCH_NUMBER = 0
            }
            TAG = "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_NUMBER}"
          }
        }
      }
    }

    stage('Set Version') {
      when {
        anyOf {
          branch 'release/*'
        }
      }
      steps {
        script {
          sh "mvn versions:set -DnewVersion=${TAG}"
          sh "mvn versions:commit"
        }
      }
    }

    stage('Compile') {
      when {
        anyOf {
          branch 'master'
          branch 'feature/*'
          branch 'release/*'
        }
      }
      steps {
        script {
          sh 'mvn compile'
        }
      }
    }

    stage('Test') {
      when {
        anyOf {
          branch 'master'
          branch 'feature/*'
          branch 'release/*'
        }
      }
      steps {
        script {
          sh 'mvn verify'
        }
      }
    }

    stage('Publish') {
      when {
        anyOf {
          branch 'master'
          branch 'release/*'
        }
      }
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            configFileProvider([configFile(fileId: "${SETTINGS_XML_FILE_ID}", variable: 'SETTINGS_XML_FILE')]) {
              sh """
                mvn deploy -s ${SETTINGS_XML_FILE} -Dusername=${USERNAME} -Dpassword=${PASSWORD}
              """
            }
          }
        }
      }
    }

    stage('Clean Up') {
      when {
        anyOf {
          branch 'release/*'
        }
      }
      steps {
        script {
          sshagent(['ssh_to_gitlab']) {
            script {
              sh "git tag ${TAG}"
              sh "git push origin tag ${TAG}"
            }
          }
        }
      }
    }
  }

  post {
    success {
      emailext body: "Your last build successed",
      subject: "SUCCESS",
      to: "${env.REGISTRY_USERNAME}"
    }

    failure {
      emailext body: "Your last build failed",
      subject: "FAILURE",
      to: "${env.REGISTRY_USERNAME}"
    }
  }
}
