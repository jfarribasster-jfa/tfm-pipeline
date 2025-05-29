
node {
    withCredentials([
        string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
        string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
        string(credentialsId: 'aws-session-token', variable: 'AWS_SESSION_TOKEN')
    ]) {
        withEnv(["AWS_REGION=us-east-1"]) {
            sh '''
                echo "Using AWS CLI with env:"
                echo "AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID"
                echo "AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY"
                echo "AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN",
                echo "AWS_REGION=us-east-1"
                echo "Verificando credenciales de AWS..."
                aws sts get-caller-identity
                echo "Iniciando sesión en ECR..."
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 186753268376.dkr.ecr.us-east-1.amazonaws.com/tfm/jenkins-agent
            '''
        }
    } 
}

pipeline {
    agent {
        docker {
            image "186753268376.dkr.ecr.us-east-1.amazonaws.com/tfm/jenkins-agent:1.0"
            reuseNode true
            alwaysPull true
        }
    }
    environment {
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "tfm-cluster-jfa"
    }
    stages {
        stage('login') {   
            steps {
                this.login()
            }
        }    
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
/**
 * Login to AWS
 */  
 def login() {
    withCredentials([
        string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
        string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
        string(credentialsId: 'aws-session-token', variable: 'AWS_SESSION_TOKEN')
    ]) {
        sh '''
            echo "Autenticando en AWS..."
            aws sts get-caller-identity

            echo "Configurando acceso a EKS..."s
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

            echo "Ejecutando kubectl..."
            kubectl get ns
            
        '''   
    }
 }                    