pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID = "574143435348"
        AWS_DEFAULT_REGION = "us-east-1"
        CLUSTER_NAME = "cluster-of-container"
        SERVICE_NAME = "node-webservice-1"
        TASK_DEFINITION_NAME = "nodejs-webserver-task-1"
        DESIRED_COUNT = "1"
        IMAGE_REPO_NAME = "repo-ecr-2"
        IMAGE_TAG = "${env.BUILD_ID}"
        REPOSITORY_URI = "public.ecr.aws/w5j7m5y6/repo-ecr-2"
        registryCredential = "aws-ecs-jenkins-cred-id"
        JOB_NAME = "aws-ecs-pipeline-job"
        TEST_CONTAINER_NAME = "${JOB_NAME}-test-server"
    }

    stages {
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Releasing') {
            steps {
                script {
                    docker.withRegistry("https://" + REPOSITORY_URI, "ecr:${AWS_DEFAULT_REGION}:" + registryCredential) {
                        dockerImage.push("${REPOSITORY_URI}:${IMAGE_TAG}")
                    }
                }

            }
        }

        stage('Deploy') {
            steps {
                withAWS(credentials: registryCredential, region: "${AWS_DEFAULT_REGION}") {
                    script {
                        sh "chmod +x -R ${env.WORKSPACE}"
                        sh './script.sh'
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -a -f'
        }
    }
}
