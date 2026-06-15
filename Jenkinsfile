pipeline {
    agent any

    environment {
        ImageRegistry = '7534286'
        EC2_IP = '3.144.43.57'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
    }

    stages {

        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh "docker build -t ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage("pushImage") {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: 'docker-login', 
                                                     passwordVariable: 'PASS', 
                                                     usernameVariable: 'USER')]) {
                        sh '''
                            echo $PASS | docker login -u $USER --password-stdin
                            docker push ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    sh """
                        # Copy files to EC2 instance
                        scp -o StrictHostKeyChecking=no ${DotEnvFile} ${DockerComposeFile} ubuntu@${EC2_IP}:/home/ubuntu/
                        
                        # Deploy using docker compose
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "
                            cd /home/ubuntu && \
                            docker compose -f ${DockerComposeFile} --env-file ${DotEnvFile} down && \
                            docker compose -f ${DockerComposeFile} --env-file ${DotEnvFile} up -d
                        "
                    """
                }
            }
        }
    }
}