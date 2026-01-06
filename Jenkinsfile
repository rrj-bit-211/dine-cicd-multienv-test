pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    BUCKET     = "dine-test-multienv"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Deploy DEV') {
      when {
        changeset "dev/**"
      }
      steps {
        withAWS(role: 'arn:aws:iam::568179491853:role/JenkinsDevDeployRole',
                region: "${AWS_REGION}") {
          sh """
            aws s3 sync dev/ s3://${BUCKET}/dev/ --delete
          """
        }
      }
    }

    stage('Approval for PROD') {
      when {
        changeset "prod/**"
      }
      steps {
        input message: "Approve deployment to PROD?"
      }
    }

    stage('Deploy PROD') {
      when {
        changeset "prod/**"
      }
      steps {
        withAWS(role: 'arn:aws:iam::568179491853:role/JenkinsProdDeployRole',
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
      echo "✅ Deployment completed successfully"
    }
    failure {
      echo "❌ Deployment failed"
    }
  }
}
