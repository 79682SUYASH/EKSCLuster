pipeline {
    agent any
    stages{
        stage('git clone'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/eks-terraform.git'
            }
        }
        stage('terraform init'){
            steps{
                withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                     sh 'terraform init'
                }
               
            }
        }
        stage('terraform plan'){
            steps{
                withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                sh 'terraform plan'
                }
            }
        }
        stage('terraform apply'){
            steps{
                withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                sh 'terraform apply --auto-approve'
                }
            }
        }
    }
}
