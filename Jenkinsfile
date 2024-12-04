pipeline {
    agent any

    environment {
        MY_ENV_FILE = credentials('MY_ENV_FILE')
        NETWORK_NAME = 'mynetwork'
        WEB_CONTAINER_NAME = 'web_container'
        //WEB_IMAGE_NAME = '20221174/ci-cd:${env.BUILD_ID}'
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
                    // Docker 이미지 빌드
                    def myapp = docker.build("20221174/ci-cd:${env.BUILD_ID}")
                    
                    // Docker Hub에 푸시
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        myapp.push("latest")  // latest 태그 푸시
                        myapp.push("${env.BUILD_ID}")  // 빌드 넘버 태그 푸시
                    }
                    
                    // Docker Hub 로그아웃
                    sh 'docker logout'
                }
            }
}

        stage('Deploy to GKE') {
            steps {
                script {
                    sh "sed -i 's|{{IMAGE_TAG}}|${env.BUILD_ID}|g' deployment.yaml"
                    step([$class: 'KubernetesEngineBuilder', 
                          projectId: env.PROJECT_ID, 
                          clusterName: env.CLUSTER_NAME,
                          location: env.LOCATION, 
                          manifestPattern: 'deployment.yaml', 
                          credentialsId: env.CREDENTIALS_ID,
                          verifyDeployments: true])
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
