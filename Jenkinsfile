// Importing necessary libraries
@Library('jenkins-utils@main')_
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
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 726122148382.dkr.ecr.us-east-1.amazonaws.com/tfm/jenkins-agent
            '''
        }
    }
    checkout scm
    def repoUrl = env.GITHUB_REPO_GIT_URL?.replaceFirst('git://', 'https://') ?: 'https://github.com/user/repo.git'
    def branch = env.CHANGE_TARGET ?: 'main'
    checkout ([
        $class: 'GitSCM',
        branches: [[name: "*/${branch}"]],
        userRemoteConfigs: [[url: repoUrl, credentialsId: 'GitHubUser']]
    ])
    datas = readYaml file: 'build.yml'  
}

pipeline {
    agent {
        docker {
            image "726122148382.dkr.ecr.us-east-1.amazonaws.com/tfm/jenkins-agent:1.0"
            args '-v /var/run/docker.sock:/var/run/docker.sock'
            reuseNode true
            alwaysPull true
        }
    }
    environment {
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "tfm-cluster-jfa"
        SONAR_HOST_URL = "http://ec2-54-210-207-177.compute-1.amazonaws.com:9000"
        ECR = "726122148382.dkr.ecr.us-east-1.amazonaws.com/tfm/"
    }
    stages {
        stage ('Checkout Code') {
            steps {
                script {
                    def repoUrl = env.GITHUB_REPO_GIT_URL?.replaceFirst('git://', 'https://') ?: 'https://github.com/user/repo.git'
                    def branch = env.CHANGE_TARGET ?: 'main'
                    echo 'Checking out code...'
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/main"]],
                        userRemoteConfigs: [[url: 'https://github.com/jfarribasster-jfa/platform-src-utils.git', credentialsId: 'GitHubUser']],
                        extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'tools'],
                                     [$class: 'CleanBeforeCheckout']]
                    ])
                    checkout ([
                        $class: 'GitSCM',
                        branches: [[name: "*/${branch}"]],
                        userRemoteConfigs: [[url: repoUrl, credentialsId: 'GitHubUser']],
                        extensions: [[$class: 'CleanBeforeCheckout']]
                    ])
                }
            }
        }
        stage('login') {   
            steps {
                echo 'login...'
                this.login()
            }
        }
        stage ('Unit Tests') {
            steps {
                echo 'Running unit tests...' 
                sh '''
                    echo "Creando entorno virtual..."
                    python3 -m venv venv
                    . venv/bin/activate
                    echo "Actualizando pip e instalando dependencias de test..."
                    pip install --upgrade pip
                    pip install -r app/requirements_test.txt

                    echo "Ejecutando pruebas unitarias..."
                    mkdir -p reports
                    cd app && pytest tests \
                        --junitxml=../reports/test-results.xml \
                        --cov=. \
                        --cov-report=xml:../reports/coverage.xml
                '''
            }
        }
        stage('Static Code Analysis') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }   
            steps {
                echo 'Running static code analysis...'
                this.static_code_analysis()
            }
        }    
        stage('Build and Push') {
            when {
                allOf{
                    // Only build and push if datas object is not null
                    expression { datas.phases.build.dockerfile != null }
                }
            }
            steps {
                echo 'Running build and push...'
                this.build_push()
            }
        }
        stage('Deploy') {
            when {
                allOf{
                    // Only deploy if datas object is yes
                    expression { "${datas.phases.deploy.enabled}" == "yes" }
                }
            }   
            steps {
                echo 'Deploying...'
                this.deploy()
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
        // Extrae el nombre del repo desde la URL de origen
        def repoUrl = env.GITHUB_REPO_GIT_URL?: 'https://github.com/user/repo.git' 
        def repoName = repoUrl?.tokenize('/').last()?.replace('.git', '') ?: 'default-project'
        def dockerConfig = '/tmp/.docker'
        def kubeHome = '/tmp/kubehome'
        sh """
            echo "Autenticando en AWS..."
            export DOCKER_CONFIG=${dockerConfig}
            export HOME=${kubeHome}
            mkdir -p \$DOCKER_CONFIG
            mkdir -p ${kubeHome}/.kube
            aws sts get-caller-identity
            
            echo "Iniciando sesión en ECR..."
            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR}${repoName}

            echo "Configurando acceso a EKS..."
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

            echo "Ejecutando kubectl..."
            kubectl get ns
            kubectl create namespace rsvpapp || echo "Namespace rsvpapp ya existe"
            aws ecr get-login-password --region us-east-1 | kubectl create secret docker-registry regcred --docker-server=726122148382.dkr.ecr.us-east-1.amazonaws.com --docker-username=AWS --docker-password="\$(cat -)" --namespace rsvpapp
        """  
    }
 }                 
/**
 * Static Code Analysis using SonarQube
 */ 
 def static_code_analysis() {
    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
    withSonarQubeEnv('SonarQubeUnir') {    
            withCredentials([
                string(credentialsId: 'SonarQubeToken', variable: 'SONAR_TOKEN')
                ]) { 
                    // Extrae el nombre del repo desde la URL de origen
                    def repoUrl = env.GITHUB_REPO_GIT_URL?: 'https://github.com/user/repo.git' 
                    def repoName = repoUrl?.tokenize('/').last()?.replace('.git', '') ?: 'default-project'
                    sh """
                        echo "Iniciando análisis de código estático..."
                        sonar-scanner -v
                        sonar-scanner \
                            -Dsonar.projectKey=${repoName} \
                            -Dsonar.sources=app \
                            -Dsonar.exclusions=**/venv/**,**/__pycache__/**,**/*.pyc,**/tests/** \
                            -Dsonar.python.coverage.reportPaths=reports/coverage.xml \
                            -Dsonar.junit.reportPaths=reports/test-results.xml \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_TOKEN
                    """
                }
            } 
    }
 }

 /**
 * Build and push step definition
 */
def build_push () {
   withCredentials([
        string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
        string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
        string(credentialsId: 'aws-session-token', variable: 'AWS_SESSION_TOKEN')
    ]) {
        def map = datas.phases.build.dockerfile
        def dockerConfig = '/tmp/.docker'
        // Extrae el nombre del repo desde la URL de origen
        def repoUrl = env.GITHUB_REPO_GIT_URL?: 'https://github.com/user/repo.git' 
        def repoName = repoUrl?.tokenize('/').last()?.replace('.git', '') ?: 'default-project'      
        // looping map containing dockerfile and version
        sh """
            echo "Autenticando en AWS..."
            export DOCKER_CONFIG=${dockerConfig}
            mkdir -p \$DOCKER_CONFIG
            echo "Iniciando sesión en ECR..."
            aws sts get-caller-identity 
            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR}${repoName}    
        """
        map.each{k, v -> 
            buildAndPush("${ECR}","${k}","${v}","${datas.phases.build.cache}")
        }
    }
}

/**
 * Deploy step definition
 */
def deploy() {
    sh "chmod +x tools/*.sh"
    sh "cp tools/*.sh ${datas.phases.deploy.path}"
    // Extrae el nombre del repo desde la URL de origen
    def repoUrl = env.GITHUB_REPO_GIT_URL?: 'https://github.com/user/repo.git' 
    def repoName = repoUrl?.tokenize('/').last()?.replace('.git', '') ?: 'default-project'
    def branch = env.CHANGE_TARGET ?: 'main'
    def KV_PATH = "${repoName}/${branch}"
    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
        withCredentials([
            string(credentialsId: 'vault_adminKV', variable: 'SECRETID'),
            string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
            string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
            string(credentialsId: 'aws-session-token', variable: 'AWS_SESSION_TOKEN')
        ]) {
            script {
                ROLE_ID="90e0490d-252d-d60a-bc55-c9a96f4edb2c"
                  def kubeHome = '/tmp/kubehome'
                // Deploy manifest files
                dir("${datas.phases.deploy.path}") {
                    def files = datas.phases.deploy.files
                    files.each { file ->
                        echo "Applying manifest k8s file: ${file}"
                        sh """
                            ./replace-secrets.sh ${KV_PATH} ${file} ${ROLE_ID} ${SECRETID}
                            export HOME=${kubeHome}
                            aws sts get-caller-identity
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                            kubectl apply -f ./${file} 
                        """
                    }   
                }
            }
        }
    }
}