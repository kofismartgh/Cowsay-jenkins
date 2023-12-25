pipeline {
    agent any
    stages {
        stage('checkout SCM') {
            steps {
                checkout scm: scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'gitlab_ssh', url: 'git@gitlab.com:kofi2454651/Cowsay.git']])
            }
        }
        stage('build') {
            steps {
                echo 'Running my docker image build.'
                sh "docker build -t cowsay:${BUILD_NUMBER} ."
                sh "docker tag cowsay:${BUILD_NUMBER} cowsay:latest"
                sh "docker image ls | grep cowsay"

            }
        }
        stage('running') {
            steps {
                echo "Running container" 
                sh "docker run -d --name cowsay-${BUILD_NUMBER} -p9000:8080 cowsay:${BUILD_NUMBER}"
            }
        }
        stage('health_check') {
            steps {
                echo "Waiting for the container to start..."
                sh "sleep 10s"

            }
        }
        stage('Regression tests') {
            steps {
                sh 'echo Running E2E tests'
            }
        }
        stage('Release to prod') {
            steps {
                sh 'echo Releasing to prod'
            }
        }
    }
 post {
        always {
            echo 'Cleanup after everything!'
        }
    }
}
