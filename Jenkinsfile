pipeline {
  agent any
  environment {
    BUILTIMAGE = 'hello-world'
    ECRURL = 'https://644832730935.dkr.ecr.us-gov-west-1.amazonaws.com'
    ECRCRED = 'ecr:us-gov-west-1:svc-jenkins'
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  stages {
    stage('Preparation') {
      steps {
        // for display purposes
        echo "Preparing"
      }
    }
    stage('Build Prep') {
      steps {
        script
          {
            // calculate GIT lastest commit short-hash
            gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            shortCommitHash = gitCommitHash.take(7)
            // calculate a sample version tag
            VERSION = shortCommitHash
            // set the build display name
            currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
          }
        }
    }
    stage('Docker Build') {
      steps {
          script {
            sh "docker build --no-cache -t ${BUILTIMAGE}:${VERSION} . "
          }
      }
    }
      stage('Scan') {
        steps {
            script {
          twistlockScan ca: '', cert: '', compliancePolicy: 'warn', \
         dockerAddress: 'unix:///var/run/docker.sock', \
         ignoreImageBuildTime: true, key: '', logLevel: 'true', \
         policy: 'warn', repository: BUILTIMAGE, \
         requirePackageUpdate: false, tag: VERSION, timeout: 10
      }
    }
    }
      stage('Publish to Twistlock') {
        steps {
          script {
          twistlockPublish ca: '', cert: '', \
            dockerAddress: 'unix:///var/run/docker.sock', key: '', \
              logLevel: 'true', repository: BUILTIMAGE, tag: VERSION, \
                timeout: 10
        }
      }
      }
      stage('Publish') {
        steps {
          script
            {
              docker.withRegistry(ECRURL, ECRCRED)
            {
                docker.image(BUILTIMAGE).push(VERSION)
            }
          }
        }
      }
    }
  }
