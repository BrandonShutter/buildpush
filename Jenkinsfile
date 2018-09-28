pipeline

{

    options

    {

        buildDiscarder(logRotator(numToKeepStr: '3'))

    }

    agent any

    environment

    {

        VERSION = 'latest'

        PROJECT = 'hello-world'

        IMAGE = 'hello-world:latest'

        ECRURL = 'https://644832730935.dkr.ecr.us-gov-west-1.amazonaws.com'

        ECRCRED = 'ecr:us-gov-west:24902970-0840-4f85-bef2-a37dbd1883c0'

    }

    stages

    {

        stage('Build preparations')

        {

            steps

            {

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

        stage('Docker build')

        {

            steps

            {

                script

                {

                    // Build the docker image using a Dockerfile

                    docker.build("$IMAGE")

                }

            }

        }

        stage('Docker push')

        {

            steps

            {

                script

                {

                    // login to ECR - for now it seems that that the ECR Jenkins plugin is not performing the login as expected. I hope it will in the future.

                    sh("eval \$(aws ecr get-login --no-include-email --region us-gov-west-1 | sed 's|https://||')")

                    // Push the Docker image to ECR

                    docker.withRegistry(ECRURL, ECRCRED)

                    {

                        docker.image(IMAGE).push()

                    }

                }

            }

        }

    }



    post

    {

        always

        {

            // make sure that the Docker image is removed

            sh "docker rmi $IMAGE | true"

        }

    }

}
