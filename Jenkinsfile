pipeline {
    agent { label "master" }
    environment {
        ECR_REGISTRY="547186126689.dkr.ecr.us-east-1.amazonaws.com"
        APP_REPO_NAME="yurttav/phonebook"
        GRANDMASTER= sh(script: 'aws ec2 describe-instances --filters Name=tag:Name,Values="Docker Control Machine"  Name=instance-state-name,Values=running --region "us-east-1" --query "Reservations[].Instances[].InstanceId" --output text', returnStdout: true)
    }
    stages {

        stage('Deploy') {
            steps {
                script {
                    env.COMMAND = sh(script: '/usr/local/bin/mssh $GRANDMASTER --region us-east-1 -oStrictHostKeyChecking=no "docker swarm join-token manager | grep \"token\""', returnStdout: true)
                }

                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_REGISTRY'
                sh 'docker pull $ECR_REGISTRY/$APP_REPO_NAME:latest'
                sh '$COMMAND'
                sh 'docker stack deploy -c docker-compose.yml phonebook'                
                    
            }
        }

    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}