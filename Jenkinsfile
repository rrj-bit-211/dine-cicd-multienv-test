pipeline {
    agent any

    paramet
    {
        choice(
            name: 'BRANCH',
            choices: ['dev_branch', 'prod_branch'],
            description: 'Select the environment to deploy to'
        )
    }

    environment {
        AWS_REGION = 'ap-south-1'
        BUCKET     = 'dine-test-multienv'
    }

    stages {

        stage('Checkout selected branch') {
            steps {
                [$class: 'GitSCM',
          branches: [[name: "${BRANCH}"]],
          userRemoteConfigs: [[url: 'https://github.com/rrj-bit-211/dine-cicd-multienv-test.git']]]
            }
        }

        stage('Deploy DEV') {
              when {
                expression { "${BRANCH}" == 'dev_branch' }
              }
            steps {
                withAWS(credentials: 'aws-poc-creds',
                role: 'arn:aws:iam::568179491853:role/JenkinsDevDeployRole',
                region: "${AWS_REGION}") {
                    sh """
            aws s3 sync dev/ s3://${BUCKET}/dev/ --delete
          """
                }
            }
        }

        // stage('Approval + Deploy PROD') {
        //     agent none
        //     when {
        //         changeset 'prod/**'
        //     }
        //     steps {
        //         timeout(time: 10, unit: java.util.concurrent.TimeUnit.SECONDS) {
        //             echo 'Skipping wait for approval in test'
        //         }
        //         input message: 'Approve deployment to PROD?'
        //     }
        // }

        stage('Approval + Deploy PROD') {
            // when {
            //     branch name: 'prod*', comparator: 'ANT'
            // }  //when branch is prod_branch
              when {
                expression { "${BRANCH}" == 'prod_branch' }
              }
            steps {
                input message: 'Approve deployment to PROD?'
                withAWS(credentials: 'aws-poc-creds',
                role: 'arn:aws:iam::568179491853:role/JenkinsProdDeployRole',
                region: "${AWS_REGION}") {
                    sh """
            aws s3 sync prod/ s3://${BUCKET}/prod/ --exact-timestamps
          """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully'
        }
        failure {
            echo '❌ Deployment failed'
        }
    }
}
