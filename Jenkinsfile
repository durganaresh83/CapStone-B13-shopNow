pipeline {
  agent any

  environment {
    DOCKERHUB_USER = "your-dockerhub-username"   // 🔁 CHANGE THIS
    FE_IMAGE = "durga/shopnow-frontend"
    BE_IMAGE = "durga/shopnow-backend"
    ADMIN_IMAGE= "durga/shopnow-admin"
    IMAGE_TAG = "${BUILD_NUMBER}"

    CLUSTER_NAME = "durga-shopnow"   // 🔁 Your actual EKS cluster name
    AWS_REGION = "eu-west-2"
  }

  stages {

    stage("Checkout") {
      steps {
        git branch: 'feature/assignment', url: 'https://github.com/durganaresh83/CapStone-B13-shopNow.git'
      }
    }

    stage("Build Images") {
      steps {
        sh """
          docker build -t $FE_IMAGE:$IMAGE_TAG frontend/
          docker build -t $BE_IMAGE:$IMAGE_TAG backend/
          docker build -t $ADMIN_IMAGE:$IMAGE_TAG admin/
        """
      }
    }

    stage("Login to DockerHub") {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
          )
        ]) {
          sh """
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
          """
        }
      }
    }

    stage("Push Images") {
      steps {
        sh """
          docker push $FE_IMAGE:$IMAGE_TAG
          docker push $BE_IMAGE:$IMAGE_TAG
          docker push $ADMIN_IMAGE:$IMAGE_TAG
        """
      }
    }

    stage("Deploy to EKS with Helm") {
      steps {
        sh """
          echo "Updating kubeconfig..."
          aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

          echo "Deploying with Helm..."
          helm upgrade --install shopnow ./shopnow \
            --set image.frontend.repository=$FE_IMAGE \
            --set image.backend.repository=$BE_IMAGE \
            --set image.backend.repository=$ADMIN_IMAGE \
            --set image.frontend.tag=latest \
            --set image.backend.tag=latest \
            --set image.admin.tag=latest \
            -n shopnow
        """
      }
    }
  }

  post {
    success {
      echo "🚀 DockerHub build & EKS deployment successful!"
    }
    failure {
      echo "❌ Pipeline failed! Check logs above."
    }
  }
}