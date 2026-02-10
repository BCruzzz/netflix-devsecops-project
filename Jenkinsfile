pipeline {
    agent any

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
                echo "Cloning repository from GitHub..."
                checkout scm
            }
        }

        stage("Install Dependencies") {
            steps {
                echo "Installing project dependencies..."
                sh "npm install"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                echo "Running SonarQube Scan..."

                withSonarQubeEnv("SonarQube") {
                    sh """
                      sonar-scanner \
                        -Dsonar.projectKey=Netflix \
                        -Dsonar.projectName=Netflix \
                        -Dsonar.host.url=http://15.228.236.43:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage("Docker Build") {
            steps {
                echo "Building Docker image..."

                sh """
                  docker build \
                    --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY \
                    -t ${IMAGE_NAME} .
                """
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

        stage("Deploy to EKS") {

            environment {
                AWS_ACCESS_KEY_ID     = credentials("aws-access-key")
                AWS_SECRET_ACCESS_KEY = credentials("aws-secret-key")
            }

            steps {
                echo "Deploying application to AWS EKS..."

                sh """
                  aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}

                  echo "Kubernetes Cluster Nodes:"
                  kubectl get nodes

                  echo "Applying Kubernetes manifests..."
                  kubectl apply -f k8s/

                  echo "Waiting for rollout..."
                  kubectl rollout status deployment netflix-deployment
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }

        failure {
            echo "❌ Pipeline failed. Check the console logs above."
        }
    }
}
