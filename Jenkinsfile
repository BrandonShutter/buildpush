pipeline {
  agent { label 'ecs-slave' }
  environment {
    VERSION = 'latest'
    PROJECT = 'hello-world'
    IMAGE = 'ubuntu:latest'
    BUILTIMAGE = 'ubuntutest'
    REPO = 'dev/ubuntu'
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
        IMAGE = "$PROJECT:$VERSION"
      }

      }
    }
    stage('Build') {
      steps {
        sh 'echo "FROM ($IMAGE) > Dockerfile'
        sh 'echo "MAINTAINER Brandon Shutter <brandon.p.shutter@nasa.gov>" >> Dockerfile'
        sh 'echo "RUN mkdir -p /tmp/test/dir" >> Dockerfile'
        sh 'docker build --no-cache -t ($BUILTIMAGE) .'
      }
    }
      stage('Scan') {
        steps {
          twistlockScan ca: '', cert: '', compliancePolicy: 'warn', \
            dockerAddress: 'unix:///var/run/docker.sock', \
            ignoreImageBuildTime: false, key: '', logLevel: 'true', \
            policy: 'warn', repository: REPO, \
            requirePackageUpdate: false, tag: 'test', timeout: 10
      }
              }

      stage('Publish to Twistlock') {
        steps {
          twistlockPublish ca: '', cert: '', \
            dockerAddress: 'unix:///var/run/docker.sock', key: '', \
            logLevel: 'true', repository: REPO, tag: 'test', \
            timeout: 10
            }
      }
    }
    stage('Publish') {
      steps {
        script
        {
            // login to ECR - for now it seems that that the ECR Jenkins plugin is not performing the login as expected. I hope it will in the future.
            // sh("eval \$(aws ecr get-login --no-include-email --region us-gov-west-1 | sed 's|https://||')")
            // Push the Docker image to ECR
            docker.withRegistry(ECRURL, ECRCRED)
            {
                docker.image(BUILTIMAGE).push()
            }
          }
        }
      }
    }
