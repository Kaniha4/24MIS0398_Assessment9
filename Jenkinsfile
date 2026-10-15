pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'kaniha4/student-placement-portal'
        DOCKER_EXE = 'C:\\Users\\91822\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Users\\91822\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        NODE_PORT = '30014'
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kaniha4/24MIS0398_Assessment9.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" buildx build --load -t %DOCKER_IMAGE%:v1 .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub2',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                        "%DOCKER_EXE%" login -u "%DOCKER_USER%" -p "%DOCKER_PASS%"
                        "%DOCKER_EXE%" push %DOCKER_IMAGE%:v1
                        "%DOCKER_EXE%" logout
                    '''
                }
            }
        }

        stage('Deploy and Verify') {
            steps {
                withCredentials([file(
                    credentialsId: 'kuberconfig',
                    variable: 'KUBECONFIG_FILE'
                )]) {
                    bat '''
                        set "KUBECONFIG=%KUBECONFIG_FILE%"
                        "%KUBECTL_EXE%" apply -f deployment.yaml
                        "%KUBECTL_EXE%" rollout status deployment/placement-portal --timeout=120s
                        if errorlevel 1 exit /b 1
                        "%KUBECTL_EXE%" get pods
                        "%KUBECTL_EXE%" get service placement-portal-service
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Student Placement Portal deployed successfully!'
        }
        failure {
            echo 'Deployment failed. Check Console Output.'
        }
    }
}
