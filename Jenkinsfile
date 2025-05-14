
node {
    withCredentials([
        string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
        string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
    ]) {
        withEnv(["AWS_REGION=us-east-1"]) {
            sh '''
                echo "Using AWS CLI with env:"
                echo "AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID"
                echo "AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY"
                aws sts get-caller-identity
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 497577049231.dkr.ecr.us-east-1.amazonaws.com
            '''
        }
    } 
}

pipeline {
    agent {
        docker {
            image "497577049231.dkr.ecr.us-east-1.amazonaws.com/tfm/jenkins-agent:1.0"
            args "-u jenkins --entrypoint='' -v /var/run/docker.sock:/var/run/docker.sock --security-opt seccomp=unconfined"
            reuseNode true
            alwaysPull true
        }
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
