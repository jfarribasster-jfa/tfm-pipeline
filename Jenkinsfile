pipeline {
    agent {
        docker {
            image 'ubuntu:22.04'
            reuseNode false
        }
    }
    stages {
        stage('Contenedor real') {
            steps {
                sh '''
                    echo "Confirmando entorno Docker..."
                    echo "Usuario:"
                    whoami
                    echo "Versión de sistema:"
                    cat /etc/os-release
                    echo "Proceso 1:"
                    cat /proc/1/cgroup | head -n 1
                '''
            }
        }
    }
}
