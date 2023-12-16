pipeline {
    agent any
    environment {
    AWS_ACCOUNT_ID="574143435348" // Replace with your AWS account ID
    AWS_DEFAULT_REGION="us-east-1" // Replace with your desired AWS region
    CLUSTER_NAME="cluster-of-container" // Replace with your ECS cluster name
    SERVICE_NAME="node-webservice-1" // Replace with your ECS service name
    TASK_DEFINITION_NAME="nodejs-webserver-task-1" // Replace with your ECS task definition name
    DESIRED_COUNT="1" // Set desired count if you need to scale your service
    IMAGE_REPO_NAME="repo-ecr-2" // Replace with your ECR repository name
    IMAGE_TAG="${env.BUILD_ID}" // This is set dynamically to tag your image with the Jenkins build ID
    REPOSITORY_URI = "574143435348.dkr.ecr.us-east-1.amazonaws.com/repo-ecr-2" // Combined URI for your ECR repository
    registryCredential = "aws-ecs-jenkins-cred-id" // The Jenkins credential ID for AWS
    JOB_NAME = "aws-ecs-pipeline-job" // Your Jenkins job name
    TEST_CONTAINER_NAME = "${JOB_NAME}-test-server" // A name for the test container
}
    

   
    stages {

    // Building Docker image
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
     }
    // Run container locally and perform tests
    stage('Running tests') {
      steps{
        sh 'docker run -i --rm --name "${TEST_CONTAINER_NAME}" "${IMAGE_REPO_NAME}:${IMAGE_TAG}" npm test -- --watchAll=false'
      }
    }

    // Uploading Docker image into AWS ECR
    stage('Releasing') {
     steps{  
         script {
			docker.withRegistry("https://" + REPOSITORY_URI, "ecr:${AWS_DEFAULT_REGION}:" + registryCredential) {
                    	dockerImage.push()
            }
         }
       }
     }

    // Update task definition and service running in ECS cluster to deploy
    stage('Deploy') {
     steps{
            withAWS(credentials: registryCredential, region: "${AWS_DEFAULT_REGION}") {
                script {
			sh "chmod +x -R ${env.WORKSPACE}"
			sh './script.sh'
                }
            } 
         }
       }
     }
   // Clear local image registry. Note that all the data that was used to build the image is being cleared.
   // For different use cases, one may not want to clear all this data so it doesn't have to be pulled again for each build.
   post {
       always {
       sh 'docker system prune -a -f'
     }
   }
 }
