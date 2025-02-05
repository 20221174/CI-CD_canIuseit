# CI/CD_Can I Use It ?
team - 
project - can I use it ?

<br/>

# 시연 영상
[시연 영상 1](https://youtu.be/lgLnvRLSWRs)
<br />
[시연 영상 2](https://youtu.be/ILNLP22bFAo)

<br />

# 준비 사항
- 데이터베이스 컨테이너와 디플로이먼트, 서비스가 필요합니다.
  - 각 컨테이너와 디플로이먼트에는 MYSQL_ROOT_PASSWORD=cancanii!와 MYSQL_DATABASE=canIuseit_db로 설정되어야 합니다.
  - 또 컨테이너는 Jenkinsfile에 명시된 NETWORK_NAME에 속해야 합니다.
  - 컨테이너와 서비스 이름은 반드시 db로 설정해주세요.
  - **위의 사항들을 지킨 명령어와 매니페스트를 작성했으니 미리 실행하고 코드를 수정해주세요.**
 
- Jenkins credential을 미리 설정해야 합니다.
  - Secret file에서 ID를 MY_ENV_FILE로 환경변수 파일을 올려주세요.
  - **이때 환경변수 중, DATABASE_HOST=db로 설정되어야 합니다.**
  - Google Kubernetes Plugin 설치 후 Google Service Account from private key에서 name을 mygke로 설정해주세요.


<br/>

## 데이터베이스 컨테이너 실행 명령어
[NETWORK_NAME]엔 Jenkinsfile에 명시한 네트워크 이름을 적으면 됩니다.
`docker run --name db --network [NETWORK_NAME] -e MYSQL_ROOT_PASSWORD=cancanii! -e MYSQL_DATABASE=canIuseit_db -p 3306:3306 mysql:8`

<br/>

## 데이터베이스 디플로이먼트와 서비스 매니페스트
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql-container
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "cancanii!"
        - name: MYSQL_DATABASE
          value: "canIuseit_db"
        ports:
        - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    app: mysql
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
  clusterIP: None
```
=======
