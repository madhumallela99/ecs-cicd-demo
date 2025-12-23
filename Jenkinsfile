pipeline {
  agent any

  environment {
    AWS_REGION   = "ap-south-1"
    ACCOUNT_ID   = "505342112116"
    ECR_REPO     = "ecs-demo-repo"
    CLUSTER_NAME = "ecs-demo-cluster"
    SERVICE_NAME = "ecs-demo-service"
    TASK_FAMILY  = "ecs-demo-task"
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
        sh '''
        aws ecs describe-task-definition \
          --task-definition $TASK_FAMILY \
          --query taskDefinition > taskdef.json

        cat taskdef.json | jq '
          .containerDefinitions[0].image =
          "'$ACCOUNT_ID'.dkr.ecr.'$AWS_REGION'.amazonaws.com/'$ECR_REPO':'$IMAGE_TAG'" |
          del(
            .taskDefinitionArn,
            .revision,
            .status,
            .requiresAttributes,
            .compatibilities,
            .registeredAt,
            .registeredBy
          ) > new-taskdef.json

        aws ecs register-task-definition \
          --cli-input-json file://new-taskdef.json
        '''
      }
    }

    stage("Deploy to ECS (Rolling Update)") {
      steps {
        sh '''
        aws ecs update-service \
          --cluster $CLUSTER_NAME \
          --service $SERVICE_NAME \
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
