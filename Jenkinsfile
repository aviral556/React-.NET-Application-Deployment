pipeline {
  agent any

  environment {
    ECR_REGISTRY = "your_aws_account_id.dkr.ecr.us-east-1.amazonaws.com"
    EKS_CLUSTER  = "export-genius-eks"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build React Frontend') {
      steps {
        dir('frontend') {
          sh 'npm install'
          sh 'npm run build'
          sh 'docker build -t $ECR_REGISTRY/react-app:latest .'
        }
      }
    }

    stage('Build .NET Backend') {
      steps {
        dir('backend') {
          sh 'dotnet build --configuration Release'
        }
      }
    }

    stage('Test .NET Backend') {
      steps {
        dir('backend') {
          sh 'dotnet test --no-build --verbosity normal'
        }
      }
    }

    stage('Build .NET Docker Image') {
      steps {
        dir('backend') {
          sh 'docker build -t $ECR_REGISTRY/dotnet-api:latest .'
        }
      }
    }
    
    stage('Trivy Scan') {
            steps {
                sh '''
                    if ! command -v trivy &> /dev/null
                    then
                      echo "Installing Trivy..."
                      sudo apt-get update
                      sudo apt-get install wget apt-transport-https gnupg lsb-release -y
                      wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
                      echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
                      sudo apt-get update
                      sudo apt-get install trivy -y
                    fi

                    echo "Running Trivy scan..."
                    trivy image --exit-code 1 --severity HIGH,CRITICAL $ECR_REGISTRY/dotnet-api:latest
                '''
            }
        }

    stage('Push Images to ECR') {
      steps {
        script {
          sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_REGISTRY'
          sh 'docker push $ECR_REGISTRY/react-app:latest'
          sh 'docker push $ECR_REGISTRY/dotnet-api:latest'
        }
      }
    }

    stage('Deploy to EKS') {
      steps {
        script {
          sh 'aws eks update-kubeconfig --name $EKS_CLUSTER --region us-east-1'
          sh 'kubectl apply -f k8s/'
        }
      }
    }
  }
}