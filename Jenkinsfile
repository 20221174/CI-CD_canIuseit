pipeline {
    agent any

    environment {
        MY_ENV_FILE = credentials('MY_ENV_FILE')
        NETWORK_NAME = 'mynetwork'
        WEB_CONTAINER_NAME = 'web_container'
        WEB_IMAGE_NAME = '20221174/ci-cd'
        PROJECT_ID = 'open-source-software-435607'
        CLUSTER_NAME = 'cluster'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'mygke'
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub') 
    }

    stages {
        stage('Extract Env Variables') {
            steps {
                script {
                    sh "cat ${MY_ENV_FILE} > .env"
                }
            }
        }

        stage('Build Web Container') {
            steps {
                script {
                    sh 'docker build -t $WEB_IMAGE_NAME .'
                }
            }
        }

        stage('Run Web Container') {
            steps {
                script {
                    sh "docker run -d --name $WEB_CONTAINER_NAME --network $NETWORK_NAME -p 3000:3000 $WEB_IMAGE_NAME"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                    echo "Checking container is available..."
                    docker ps
                    echo "Sending request to the server..."
                    docker logs $WEB_CONTAINER_NAME
                    RESPONSE=$(docker exec web_container curl --max-time 10 -s -w "%{http_code}" -o /dev/null http://localhost:3000)
                    if [ "$RESPONSE" -eq 200 ]; then
                        echo "Server is running properly. HTTP Status: $RESPONSE"
                    else
                        echo "Test failed! HTTP Status: $RESPONSE"
                    fi
                    '''
                }
            }
        }

         stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    sh "echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin"
                    sh 'docker push $WEB_IMAGE_NAME'
                    sh 'docker logout'
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    withCredentials([file(credentialsId: env.CREDENTIALS_ID, variable: 'KUBE_CONFIG')]) {
                        sh '''
                        # Kubeconfig 설정
                        export KUBECONFIG=$KUBE_CONFIG

                        # deployment.yaml 파일에서 이미지 태그를 동적으로 변경
                        sed -i 's|image: .*$|image: ${WEB_IMAGE_NAME}:${VERSION_TAG}|' deployment.yaml

                        # kubectl apply 명령어로 업데이트
                        kubectl apply -f deployment.yaml
                        '''
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker resources...'
            sh '''
            docker stop $WEB_CONTAINER_NAME || true
            docker rm $WEB_CONTAINER_NAME || true
            docker rmi $WEB_IMAGE_NAME || true
            '''
        }
    }
}
