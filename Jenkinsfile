pipeline {
    agent any
    environment{
        PROD_IP= '3-111-57-20'
        CURL_RESP= '200'
        HOST_IP = sh(script: "ip route | awk 'NR==1 {print \$3}'", returnStdout: true).trim()
        //CURL_RESP = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" http://${HOST_IP}:9000", returnStatus: true).trim().toInteger()
    }
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
                echo "Running container" 
                sh "docker run -d --name cowsay-${BUILD_NUMBER} -p9000:8080 cowsay:${BUILD_NUMBER}"
                sh "sleep 5s "

            }
        }
        stage('Test') {
            steps {
                echo "Waiting for the container to start..."
                sh 'docker ps --filter "name=cowsay" --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"'
                sh ("""
                    until curl --include kofismartgh.mooo.com:9000 | head -n 1 | grep '200 OK'
                    do 
                        sleep 1
                    done
                """)
            }
        }
        stage('Publish') {
            steps {
                sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 644435390668.dkr.ecr.ap-south-1.amazonaws.com"
                sh "docker tag cowsay:latest 644435390668.dkr.ecr.ap-south-1.amazonaws.com/solomon:latest"
                sh "docker push 644435390668.dkr.ecr.ap-south-1.amazonaws.com/solomon:latest"
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
        stage('Deploy') {
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
        stage('E2E test'){
            steps{
                echo "testing prod up"
                sh ("""
                    until curl --include 3.111.57.20:9000 | head -n 1 | grep '200 OK'
                    do 
                        sleep 1
                    done
                """)
            }
        }
    }
 post {
        always {
            echo 'Cleanup after everything!'
            sh " docker stop cowsay-${BUILD_NUMBER} "
            sh " docker rm -f cowsay-${BUILD_NUMBER} "
            sh 'docker images --format "{{.Repository}}:{{.Tag}}" | grep "cowsay" | xargs docker rmi'
            emailext body: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'kofismart19@gmail.com'
        }
    }
}
