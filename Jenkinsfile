pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-northeast-1'
    ECR_REPO_NAME = 'my-app-repo'
    ACCOUNT_DEV = '111122223333'
    ACCOUNT_STAGING = '222233334444'
    ACCOUNT_PROD = '333344445555'
  }

  parameters {
    string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Git 分支名稱')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script {
          if (params.BRANCH_NAME == 'main') {
            env.ENV = 'production'
            env.ACCOUNT_ID = env.ACCOUNT_PROD
          } else if (params.BRANCH_NAME == 'staging') {
            env.ENV = 'staging'
            env.ACCOUNT_ID = env.ACCOUNT_STAGING
          } else {
            env.ENV = 'dev'
            env.ACCOUNT_ID = env.ACCOUNT_DEV
          }

          env.IMAGE_TAG = "${env.BUILD_NUMBER}"
          env.ECR_URI = "${env.ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}"
        }
      }
    }

    stage('Build with Gradle') {
      steps {
        sh './gradlew clean build'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh """
          aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}
          docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} .
          docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
        """
      }
    }

    stage('Push Docker Image to ECR') {
      steps {
        sh "docker push ${ECR_URI}:${IMAGE_TAG}"
      }
    }

    stage('Terraform Init & Apply') {
      steps {
        dir("terraform/${ENV}") {
          withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "aws-${ENV}"]]) {
            sh """
              terraform init
              terraform apply -auto-approve \
                -var="image_tag=${IMAGE_TAG}" \
                -var="ecr_repo=${ECR_URI}"
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo "Deployed ${ENV} successfully with image ${IMAGE_TAG}"
    }
    failure {
      echo "Deployment failed for ${ENV}"
    }
  }
}
