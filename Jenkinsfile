pipeline {
    agent any
    environment{
        PROD_IP= '3-111-57-20'
        //CURL_RESP= '200'
        HOST_IP = sh(script: "ip route | awk 'NR==1 {print \$3}'", returnStdout: true).trim()
        //CURL_RESP = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" http://${HOST_IP}:9000", returnStatus: true).trim().toInteger()
    }
    stages {
        stage('Initialize Environment') {
            steps {
                script {
                   
                    //HOST_IP = sh(script: "ip route | awk 'NR==1 {print \$3}'", returnStdout: true).trim()
                    //CURL_RESP = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" http://${HOST_IP}:9000", returnStatus: true).trim().toInteger()

                    //env.HOST_IP = HOST_IP
                    //env.CURL_RESP = CURL_RESP.toString()
                    echo "HOST_IP: ${HOST_IP}"
                    //echo "CURL_RESP: ${CURL_RESP}"

                }
            }
        }
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
        stage ('curl check'){
            steps{
                sh 'curl --version'
                script{
                   env.CURL_RESP = sh(script: "curl -s -w '\\n%{response_code}'-u http://${HOST_IP}:9000", returnStdout: true).trim().toInteger()
                   echo "Curl response code: ${CURL_RESP}" 
                   echo "here si sthe $CURL_RESP"
                   // echo "HOST_IP: ${HOST_IP}"
                }
            }
        }
        stage('health_check') {
            steps {
                echo "Waiting for the container to start..."
                sh "sleep 10s"
                //sh """
                  //      sh /home/hostip.sh
                  //  """
                echo "Performing a curl request to the running container.to ${HOST_IP} AND ${CURL_RESP}.."
                script {

                    //env.CURL_RESP = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" http://${HOST_IP}:9000", returnStdout: true).trim().toInteger()
                    
                    echo "Curl response code: ${CURL_RESP}"
                    echo "HOST_IP: ${HOST_IP}"
                    echo "CURL_RESP: ${CURL_RESP}"

                    sh """
                        if [ ${CURL_RESP} -eq 200 ]; then
                            echo "Curl request successful: Container is up and running."
                        else
                            echo "Curl request failed: Container might not be ready or is not responding correctly. HTTP status: ${CURL_RESP}"
                            exit 1
                        fi
                    """
                echo "DONE"
                }
            }
        }
        stage('Publish') {
            steps {
                sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 644435390668.dkr.ecr.ap-south-1.amazonaws.com"
                sh "docker tag cowsay:latest 644435390668.dkr.ecr.ap-south-1.amazonaws.com/solomon:latest"
                sh "docker push 644435390668.dkr.ecr.ap-south-1.amazonaws.com/solomon:latest"
            }
        }
        stage('Release to prod') {
            steps {
                echo 'copying compose to prod'
                sh "scp -i /solomon_rgt.pem /docker-compose.yml ubuntu@ec2-${PROD_IP}.ap-south-1.compute.amazonaws.com:~/."
                echo 'ssh to prod and run docker compose'
                sh """
                        ssh -t -o StrictHostKeyChecking=no -i /solomon_rgt.pem ubuntu@ec2-${PROD_IP}.ap-south-1.compute.amazonaws.com << 'EOF'
						aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 644435390668.dkr.ecr.ap-south-1.amazonaws.com
						docker compose down -v
                        docker compose up -d
						<< 'EOF'
                    """
            }
        }
    }
 post {
        always {
            echo 'Cleanup after everything!'
            sh " docker stop cowsay-${BUILD_NUMBER} "
            sh " docker rm -f cowsay-${BUILD_NUMBER} "
            sh 'docker images --format "{{.Repository}}:{{.Tag}}" | grep "cowsay" | xargs docker rmi'
        }
    }
}
