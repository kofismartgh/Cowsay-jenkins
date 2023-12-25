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
                script {
                    def existing_containers = sh (script: 'docker ps -aq -f name=cowsay-', returnStatus: true).trim()
                    if (existing_containers) {
                        echo "Stopping and removing existing cowsay"
                        sh "docker stop ${existing_containers}"
                        sh "docker rm ${existing_containers}"
                    }
                }
                sh "docker run -d --name cowsay-${BUILD_NUMBER} -p9000:8080 cowsay:${BUILD_NUMBER}"
            }
        }
        stage('health_check') {
            steps {
                echo "Waiting for the container to start..."
                sh "sleep 10s"
                script{
                   def host_ip = sh 'ip route | awk \'NR==1 {print $3}\'', returnStdout: true.trim()
                    echo "Performing a curl request to the running container..."
                    def curl_response = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" http://${host_ip}:9000", returnStatus: true).trim().toInteger()

                    echo "Curl response code: $curl_response"

                    if (curl_response == 200) {
                        echo "Curl request successful: Container is up and running."
                    } else {
                        echo "Curl request failed: Container might not be ready or is not responding correctly. HTTP status: $curl_response"
                        error "Container not ready."
                    } 


                }

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
