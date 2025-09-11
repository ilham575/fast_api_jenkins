pipeline {
    agent {
        docker {
            image 'python:3.11'
            args '-v /var/run/docker.sock:/var/run/docker.sock --network mynet -u root'
        }
    }
    environment {
        SONARQUBE = credentials('labs_1')
    }
    stages {
        stage('Install git') {
            steps {
                sh 'apt-get update && apt-get install -y git'
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ilham575/fast_api_jenkins.git'
            }
        }
        stage('Setup venv') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                pip install pytest pytest-cov
                '''
            }
        }
        stage('Run Tests & Coverage') {
            steps {
                sh 'venv/bin/pytest --maxfail=1 --disable-warnings -q --cov=app --cov-report=xml'
            }
        }
        stage('Install SonarQube Scanner') {
            steps {
                sh '''
                apt-get update
                apt-get install -y wget unzip openjdk-21-jre
                wget -O /tmp/sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
                unzip /tmp/sonar-scanner.zip -d /opt
                export PATH=$PATH:/opt/sonar-scanner-5.0.1.3006-linux/bin
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                    sh '''
                    export PATH=$PATH:/opt/sonar-scanner-5.0.1.3006-linux/bin
                    sonar-scanner
                    '''
                }
            }
        }
        stage('Install docker CLI') {
            steps {
                sh '''
                apt-get update
                apt-get install -y docker.io
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t fastapi-app:latest .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag fastapi-app:latest $DOCKER_USER/fastapi-app:latest
                    docker push $DOCKER_USER/fastapi-app:latest
                    '''
                }
            }
        }
        stage('Deploy Container') {
            steps {
                sh 'docker run -d -p 8000:8000 fastapi-app:latest'
            }
        }
    }
    post {
        always {
            echo "Pipeline finished"
        }
    }
}