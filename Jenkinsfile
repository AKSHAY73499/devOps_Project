pipeline {
    agent any

    tools {
        nodejs 'NodeJS20'
    }

    environment {
        DOCKER_IMAGE = 'akshaykumar73499/devops-project'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/AKSHAY73499/devOps_Project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('ESLint Analysis') {
            steps {
                bat 'npx eslint src'
            }
        }

        stage('SonarCloud Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarScanner'
            }

            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    bat '''
            %SCANNER_HOME%\\bin\\sonar-scanner.bat ^
            -Dsonar.projectKey=AKSHAY73499_devOps_Project ^
            -Dsonar.organization=akshay73499 ^
            -Dsonar.sources=src ^
            -Dsonar.host.url=https://sonarcloud.io ^
            -Dsonar.token=%SONAR_TOKEN%
            '''
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                bat '''
        trivy fs --format table --output trivy-report.txt .
        '''
            }
        }

        stage('Build Application') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%'
            }
        }

        stage('Deploy to Vercel') {
            steps {
                withCredentials([string(credentialsId: 'vercel-token', variable: 'VERCEL_TOKEN')]) {
                    bat 'npm install -g vercel'
                    bat 'vercel --prod --token %VERCEL_TOKEN%'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline Completed Successfully'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
