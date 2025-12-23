pipeline {
  agent any

  environment {
    AWS_REGION   = "ap-south-1"
    ACCOUNT_ID   = "505342112116"
    ECR_REPO     = "ecs-demo-repo"
    CLUSTER_NAME = "ecs-demo-cluster"
    SERVICE_NAME = "ECS-Task-Definition-service-puxei31h"
    TASK_FAMILY  = "ECS-Task-Definition"
    IMAGE_TAG    = "${BUILD_NUMBER}"
  }

  stages {

    stage("Checkout") {
      steps {
        git branch: "main",
        url: "https://github.com/madhumallela99/ecs-cicd-demo.git"
      }
    }

    stage("Build Docker Image") {
      steps {
        sh '''
        docker build -t $ECR_REPO:$IMAGE_TAG .
        '''
      }
    }

    stage("Login to ECR") {
      steps {
        sh '''
        aws ecr get-login-password --region $AWS_REGION | \
        docker login --username AWS --password-stdin \
        $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
        '''
      }
    }

    stage("Push Image to ECR") {
      steps {
        sh '''
        docker tag $ECR_REPO:$IMAGE_TAG \
        $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG

        docker push \
        $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
        '''
      }
    }

   stage("Register New Task Definition") {
  steps {
    sh """
    sed -i 's|<IMAGE_URI>|${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}|g' taskdef.json

    aws ecs register-task-definition \
      --cli-input-json file://taskdef.json
    """
  }
}

   stage("Deploy to ECS (Rolling Update)") {
  steps {
    sh '''
    LATEST_TASK_DEF=$(aws ecs list-task-definitions \
      --family-prefix ECS-Task-Definition \
      --sort DESC \
      --query "taskDefinitionArns[0]" \
      --output text | head -n 1)

    echo "Deploying task definition: $LATEST_TASK_DEF"

    aws ecs update-service \
      --cluster ecs-demo-cluster \
      --service ECS-Task-Definition-service-puxei31h \
      --task-definition $LATEST_TASK_DEF \
      --force-new-deployment
    '''
  }
}

  }

  post {
    success {
      echo "✅ ECS Rolling Update Deployment Successful"
    }
    failure {
      echo "❌ Deployment Failed – Check Logs"
    }
  }
}





