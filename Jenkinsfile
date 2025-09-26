pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_CREDENTIALS_ID = "aws-eks"   // Jenkins AWS credentials ID
        CLUSTER_NAME = "jenkins-eks-cluster"
    }

    stages {
        stage('Checkout Repo') {
            steps {
                checkout scm
            }
        }

        stage('Install Tools') {
            steps {
                sh '''
                  set -e

                  if ! command -v eksctl >/dev/null 2>&1; then
                    echo "Installing eksctl..."
                    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
                    sudo mv /tmp/eksctl /usr/local/bin
                  fi

                  if ! command -v kubectl >/dev/null 2>&1; then
                    echo "Installing kubectl..."
                    KUBECTL_VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
                    curl -LO "https://dl.k8s.io/release/${KUBECTL_VER}/bin/linux/amd64/kubectl"
                    chmod +x kubectl && sudo mv kubectl /usr/local/bin/
                  fi

                  if ! command -v aws >/dev/null 2>&1; then
                    echo "Installing AWS CLI v2..."
                    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
                    unzip -q /tmp/awscliv2.zip -d /tmp && sudo /tmp/aws/install
                  fi
                '''
            }
        }

        stage('Create EKS Cluster') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: "${AWS_DEFAULT_REGION}") {
                    sh '''
                      set -e
                      echo "Creating EKS Cluster: ${CLUSTER_NAME} in ${AWS_DEFAULT_REGION}"

                      eksctl create cluster \
                        --name ${CLUSTER_NAME} \
                        --region ${AWS_DEFAULT_REGION} \
                        --nodes 2 \
                        --nodes-min 2 \
                        --nodes-max 4 \
                        --node-type t3.medium \
                        --managed
                    '''
                }
            }
        }

        stage('Update kubeconfig & Verify') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: "${AWS_DEFAULT_REGION}") {
                    sh '''
                      echo "Updating kubeconfig..."
                      aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${CLUSTER_NAME}

                      echo "Verifying cluster nodes..."
                      kubectl get nodes
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed. Check Jenkins logs and AWS CloudFormation stacks for details.'
        }
        success {
            echo '✅ EKS Cluster created successfully!'
        }
    }
}
