pipeline {
    agent any
    environment {
        AWS_REGION = 'us-west-2' 
    }

    stages {
        stage('Set AWS Credentials') {
            steps {
                script {
                    try {
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'AWS_ACCESS_KEY'  // Updated with your credentials ID
                        ]]) {
                            sh '''
                            echo "AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID"
                            aws sts get-caller-identity
                            '''
                        }
                        echo 'Set AWS Credentials - SUCCESS'
                    } catch (Exception e) {
                        echo "Set AWS Credentials - FAILED"
                        currentBuild.result = 'FAILURE'
                        throw e  // Rethrow the exception to stop the pipeline
                    }
                }
            }
        }
        
        stage('Checkout Code') {
            steps {
                script {
                    try {
                        git branch: 'main', url: 'https://github.com/CameronLCleveland/AWS-JenkinsRepo-S3.git'
                        echo 'Checkout Code - SUCCESS'
                    } catch (Exception e) {
                        echo 'Checkout Code - FAILED'
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        
        stage('Initialize Terraform') {
            steps {
                script {
                    try {
                        sh '''
                        terraform init
                        '''
                        echo 'Initialize Terraform - SUCCESS'
                    } catch (Exception e) {
                        echo 'Initialize Terraform - FAILED'
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        
        stage('Plan Terraform') {
            steps {
                script {
                    try {
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'AWS_ACCESS_KEY'
                        ]]) {
                            sh '''
                            export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                            export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                            terraform plan -out=tfplan
                            '''
                        }
                        echo 'Plan Terraform - SUCCESS'
                    } catch (Exception e) {
                        echo 'Plan Terraform - FAILED'
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        
        stage('Apply Terraform') {
            steps {
                script {
                    try {
                        input message: "Approve Terraform Apply?", ok: "Deploy"
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'AWS_ACCESS_KEY'
                        ]]) {
                            sh '''
                            export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                            export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                            terraform apply -auto-approve tfplan
                            '''
                        }
                        echo 'Apply Terraform - SUCCESS'
                    } catch (Exception e) {
                        echo 'Apply Terraform - FAILED'
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        
        stage ('Destroy Terraform') {
            steps {
                script {
                    try {
                        input message: "Do you want to destroy the infrastructure?", ok: "Destroy"
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'AWS_ACCESS_KEY'
                        ]]) {
                            sh '''
                            export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                            export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                            terraform destroy -auto-approve
                            '''
                        }
                        echo 'Destroy Terraform - SUCCESS'
                    } catch (Exception e) {
                        echo 'Destroy Terraform - FAILED'
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Terraform deployment completed successfully!'
            // Add notification here (e.g., Slack, Email)
        }
        failure {
            echo 'Terraform deployment failed!'
            // Add notification here (e.g., Slack, Email)
        }
    }
}
