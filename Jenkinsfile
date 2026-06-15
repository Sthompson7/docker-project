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
                    echo "=== Deploy Stage - Connection Test ==="
                    sh """
                        echo "Target IP: ${EC2_IP}"
                        echo "Testing SSH connection..."
                        
                        ssh -o StrictHostKeyChecking=no -o ConnectTimeout=15 -v ubuntu@${EC2_IP} 'echo ✅ SSH Connected' || echo "❌ SSH Failed"
                    """
                }
            }
        }
}