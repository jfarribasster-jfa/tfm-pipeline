pipeline {
    agent {
        docker {
            image "497577049231.dkr.ecr.us-east-1.amazonaws.com/tfm/jenkins-agent:1.0"
            args "-v /var/run/docker.sock:/var/run/docker.sock --security-opt seccomp=unconfined"
            reuseNode false
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
                //this.login()
                sh '''
                    echo "Verificando entorno:"
                    ls -lrt
                    pwd
                '''
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

            echo "Configurando acceso a EKS..."
            ls -la
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

            echo "Ejecutando kubectl..."
            kubectl get ns
            
        '''   
    }
 }                    