# AWS Lambda Deployment with Jenkins Pipeline

This guide outlines the steps to deploy an AWS Lambda function using Jenkins Pipeline. Follow this step-by-step guide to set up and automate your deployments.

---

## Prerequisites

1. **AWS Account**:
   - Ensure you have access to an AWS account.
   - Create an IAM user with programmatic access and appropriate permissions for Lambda and S3.

2. **Jenkins Server**:
   - A running Jenkins server with access to the internet.
   - Install required Jenkins plugins:
     - AWS Credentials
     - Pipeline: AWS Steps
     - Pipeline
     - AWS Lambda

3. **AWS CLI**:
   - Install and configure the AWS CLI on the Jenkins server.

4. **Lambda Function**:
   - Your AWS Lambda function code should be packaged as a ZIP file.
   - Place the ZIP file in a Git repository or accessible storage.

5. **S3 Bucket**:
   - Create an S3 bucket to store the Lambda deployment package if needed.

6. **Jenkins Credentials**:
   - Add AWS credentials (Access Key and Secret Key) in Jenkins credentials.

---

## Jenkins Pipeline Configuration

### Step 1: Create a New Jenkins Pipeline Job

1. Open Jenkins and click on `New Item`.
2. Select `Pipeline` and name your job (e.g., `AWS Lambda Deploy`).
3. Click `OK` to create the job.

### Step 2: Define the Pipeline Script

Use the following example to define your pipeline script. Adjust paths and parameters according to your Lambda function setup.

```groovy
pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_CREDIENTIAL_INFO') // Jenkins credential ID
        AWS_SECRET_ACCESS_KEY = credentials('AWS_CREDIENTIAL_INFO')
        AWS_REGION = 'us-east-1' // Change as needed
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/arifhossen/aws-lambda.git'
            }
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh 'pip install -r requirements.txt -t .'
        //     }
        // }
        stage('Package Lambda') {
            steps {
                sh '''
                zip -r lambda-package.zip .
                '''
            }
        }
        stage('Deploy to AWS Lambda') {
            steps {
                sh '''
                aws lambda update-function-code \
                --function-name aws-infra-deploy-dev-chatbot-lambda \
                --zip-file fileb://lambda-package.zip \
                --region $AWS_REGION
                '''
            }
        }
    }
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
```

---

## Additional Steps

### Step 3: Configure AWS Credentials in Jenkins

1. Navigate to `Manage Jenkins > Credentials > System > Global credentials`.
2. Add a new credential with:
   - **Kind**: AWS Credentials
   - **Access Key ID**: Your AWS Access Key
   - **Secret Access Key**: Your AWS Secret Key
   - **ID**: `your-aws-credentials-id` (used in the script)

### Step 4: Trigger the Pipeline

1. Save the pipeline job.
2. Click `Build Now` to trigger the deployment.
3. Monitor the build logs for progress and success messages.

---

## Notes

1. **Testing the Lambda Function**:
   - After deployment, test the function in the AWS Lambda console to ensure it works as expected.

2. **Error Handling**:
   - Add error-handling mechanisms in the pipeline script if needed.

3. **Security**:
   - Rotate AWS credentials periodically.
   - Limit permissions of the IAM user to only those required for Lambda deployments.

---

## References

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/lambda/index.html)

---

### Happy Deploying!
