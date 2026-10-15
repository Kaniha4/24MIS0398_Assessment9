
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'kaniha4/student-placement-portal'
        DOCKER_EXE = 'C:\\Users\\91822\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        BUILDX_EXE = 'C:\\Users\\91822\\.docker\\cli-plugins\\docker-buildx.exe'
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
                bat '"%BUILDX_EXE%" build --load -t %DOCKER_IMAGE%:v1 .'
            }
        }

        stage('Run Local Container Test') {
            steps {
                bat '''
                    "%DOCKER_EXE%" rm -f placement-portal-local 2>NUL
                    "%DOCKER_EXE%" run -d --name placement-portal-local -p 8081:80 %DOCKER_IMAGE%:v1
                    if errorlevel 1 exit /b 1
                    "%DOCKER_EXE%" ps
                '''
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
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" push %DOCKER_IMAGE%:v1
                        if errorlevel 1 exit /b 1
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
                        if errorlevel 1 exit /b 1

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
            echo 'Local test URL: http://localhost:8081'
            echo 'Kubernetes NodePort: 30014'
        }
        failure {
            echo 'Pipeline failed. Check Console Output for the failing stage.'
        }
    }
}
