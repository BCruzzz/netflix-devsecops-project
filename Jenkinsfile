pipeline {
  agent any

  options {
    skipDefaultCheckout(true)  // <<<<<< IMPORTANTE: desliga o "Declarative: Checkout SCM"
    timestamps()
  }

  tools {
    nodejs "node16"
  }

  environment {
    IMAGE_NAME  = "bmcruzz1/netflix-devsecops:latest"
    AWS_REGION  = "sa-east-1"
    EKS_CLUSTER = "netflix-devsecops-cluster"
  }

  stages {

    stage("Checkout Code") {
      steps {
        echo "Checkout (clean clone)..."
        deleteDir()       // limpa workspace pra evitar repo quebrado
        checkout scm      // clona do SCM configurado no job
        sh "git rev-parse --is-inside-work-tree"
        sh "git status --porcelain || true"
      }
    }

    stage("Install Dependencies") {
      steps {
        echo "Installing dependencies with npm..."
        sh "npm install"
      }
    }

    stage("SonarQube Analysis") {
      steps {
        echo "Running SonarQube Scan..."

        script {
          // usa o nome do scanner conforme configurado em Global Tool Configuration
          def scannerHome = tool 'sonar-scanner'

          withSonarQubeEnv("SonarQube") {
            withCredentials([string(credentialsId: "Sonar-token", variable: "SONAR_TOKEN")]) {
              sh """
                ${scannerHome}/bin/sonar-scanner \
                  -Dsonar.projectKey=Netflix \
                  -Dsonar.projectName=Netflix \
                  -Dsonar.host.url=http://172.31.6.195:9000 \
                  -Dsonar.login=\$SONAR_TOKEN
              """
            }
          }
        }
      }
    }

    stage("Docker Build") {
      steps {
        echo "Building Docker image..."
        withCredentials([string(credentialsId: "TMDB_V3_API_KEY", variable: "TMDB_KEY")]) {
          sh """
            docker build \
              --build-arg TMDB_V3_API_KEY=\$TMDB_KEY \
              -t ${IMAGE_NAME} .
          """
        }
      }
    }

    stage("Push Image to DockerHub") {
      steps {
        echo "Pushing image to DockerHub..."
        withCredentials([usernamePassword(
          credentialsId: "dockerhub-creds",
          usernameVariable: "DOCKER_USER",
          passwordVariable: "DOCKER_PASS"
        )]) {
          sh """
            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
            docker push ${IMAGE_NAME}
          """
        }
      }
    }

    stage("Deploy to AWS EKS") {
      environment {
        AWS_ACCESS_KEY_ID     = credentials("aws-access-key")
        AWS_SECRET_ACCESS_KEY = credentials("aws-secret-key")
      }
      steps {
        echo "Deploying application to EKS..."
        sh """
          aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
          kubectl apply -f k8s/ --validate=false
          kubectl rollout status deployment netflix-deployment
        """
      }
    }
  }

  post {
    success { echo "✅ Pipeline executed successfully!" }
    failure { echo "❌ Pipeline failed. Check logs above." }
  }
}
