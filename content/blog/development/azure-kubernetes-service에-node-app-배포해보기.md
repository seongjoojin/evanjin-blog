---
title: Azure Kubernetes Service에 Node app 배포해보기
date: 2019-12-24 16:12:62
category: development
---

## 1. Azure Kubernetes Service(AKS) 란

완전 관리형 서비스로써 컨테이너화된 어플리케이션의 배포와 관리를 더욱 용이하게 해주는 서비스입니다.

AKS는 여러 사례를 친절하게 안내해주고 있습니다.
그 중에서도 저는 `간편하게 기존 애플리케이션 마이그레이션`로 진행해보려고 합니다.

https://azure.microsoft.com/ko-kr/solutions/architecture/migrate-existing-applications-with-aks/

## 2. Dockerizing

먼저 kubernetes에 올리기전에 dockerizing부터 진행하여야 합니다.

### 2-1. Dockerfile 생성

만약 docker가 설치되어있지 않은 상태라면 docker를 설치해주셔야 합니다.
https://docs.docker.com/install/

mac의 경우 homebrew로 아래와 같이 설치 가능합니다.

```shell
brew cask install docker
```

먼저 제가 만든 NodeJS 어플리케이션의 `Dockerfile`을 생성해줍니다.

```Dockerfile
FROM node:lts

WORKDIR /app
ADD . /app/

RUN npm i -g npm
RUN npm i && npm run build

ENV HOST 0.0.0.0
EXPOSE 4000

CMD ["npm", "run", "start"]
```

Ddockerfile은 해당 NodeJS 어플리케이션마다 다르기 때문에 구성하신 scripts나 설정마다 다르게 해주어야합니다.

docker에서 pm2 설정 하실 경우에는 pm2의 공식 홈페이지에 더욱 자세히 나와있으므로 참고하시길 바랍니다.

https://pm2.keymetrics.io/docs/usage/docker-pm2-nodejs/

Dockerfile을 만드셨다면 `.dockerignore` 파일도 생성해줍니다.

```
node_modules
package-lock.json
yarn.lock
```

저는 node_modules만 넣었지만 컨테이너에 포함되지 말아야할 것들이 있다면 추가해주시면 됩니다.

### 2-2. Docker image 생성 및 테스트

다음으로 Docker 컨테이너 이미지를 생성해줍니다.

```shell
docker build -t [이미지 이름]:[태그 이름] .

# 예시
docker build -t jintube:0.1.0 .
```

여기서 jintube 부분이 이미지의 이름이 되게 되는데요.
이부분은 여러분들이 하시고 싶은 이름으로 진행하셔야 합니다.

이 후에 테스트로 이 Docker 이미지를 실행해보도록 하겠습니다.

```shell
docker run -it -p 4000:4000 [이미지 이름]:[태그 이름]

# 예시
docker run -it -p 4000:4000 jintube:0.1.0
```

포트와 이미지 이름은 생성하신 것에 따라 다를 수 있으니 조정해서 사용하시면 됩니다.

## 3. Azure Container Registry 이용하기

Dockerizing을 완료하였다면 이제 Azure Container Registry에 만들어진 컨테이너 이미지를 올립니다.

진행하시기 전에 Azure에 접속할 계정을 만들어주셔야 합니다.
기존의 마이크로소프트 계정이 있으시다면 그 계정으로 접속해주시면 됩니다.
또한 깃허브 계정으로도 로그인이 됩니다.
만약 깃허브 계정을 연결하시기 싫으시다면 마이크로소프트 계정을 새로 만드시면 됩니다.

### 3-1. Azure CLI 설치

각 운영체제 마다 다르게 설치 가능합니다.
아래의 링크를 참조하시면 됩니다.

https://docs.microsoft.com/ko-kr/cli/azure/install-azure-cli?view=azure-cli-latest

mac의 경우에는 homebrew로 설치 가능합니다.

```shell
# 설치
$ brew install azure-cli

# 설치 확인
$ az --version
```

설치 후 로그인을 해주어야합니다.

```shell
$ az login
```

### 3-2. Azure Container Registry 만들기

먼저 리소스 그룹을 생성해줍니다.

```shell
$ az group create --name [리소스 그룹 이름] --location [지역 이름]

# 예시
$ az group create --name jintube-resource --location koreacentral
```

지역이름은 아래의 링크에서 참조하시면 됩니다.
https://azure.microsoft.com/en-us/global-infrastructure/locations/

다음으로 Azure Container Registry 인스턴스를 생성해줍니다.

```shell
$ az acr create --resource-group [리소스 그룹 이름] --name [acrName] --sku Basic

# 예시
$ az acr create --resource-group jintube-resource --name jintubearc --sku Basic
```

다음으로 Container Registry에 로그인하여야 합니다.

```shell
$ az acr login --name [acrName]

$ az acr login --name jintubearc
```

### 3-3. 컨테이너 이미지 푸시

먼저 컨테이너 이미지를 푸시하기 위해 로그인 서버 주소를 가지고 옵니다.

```shell
$ az acr list --resource-group [리소스 그룹 이름] --query "[].{acrLoginServer:loginServer}" --output table

# 예시
$ az acr list --resource-group jintube-resource --query "[].{acrLoginServer:loginServer}" --output table
```

다음으로 이미지 태그를 지정해줍니다.

```shell
$ docker tag [태그 이름] [이미지 서버 주소]/[컨테이너 이름]:[버전]

# 예시
$ docker tag jintube:0.1.0 jintubearc.azurecr.io/jintube:v1
```

태그가 제대로 적용되었는지는 `docker images` 명령으로 확인해주시면 됩니다.

이제 레지스트리에 이미지를 푸시해보겠습니다.

```shell
$ docker push [이미지 서버 주소]/[컨테이너 이름]:[버전]

# 예시
$ docker push jintubearc.azurecr.io/jintube:v1
```

푸시가 완료되었으면 제대로 ACR에 올라갔는지 한 번 확인해봅니다.

```shell
$ az acr repository list --name [acrName] --output table

# 예시
$ az acr repository list --name jintubearc --output table
```

## 4. Kubernetes 클러스터 만들기

```shell
$ az aks create \
    --resource-group [리소스 그룹 이름] \
    --name [클러스터 이름] \
    --node-count [노드 갯수] \
    --generate-ssh-keys \
    --attach-acr [acrName]

# 예시
$ az aks create \
    --resource-group jintube-resource \
    --name jintubeakscluster \
    --node-count 2 \
    --generate-ssh-keys \
    --attach-acr jintubearc
```

다음으로 로컬 컴퓨터에서 Kubernetes 클러스터를 연결하기 위해서 kubectl를 사용하여야 합니다.

아래 명령어로 설치가 가능합니다.

```shell
$ az aks install-cli
```

다음으로 kubectl를 사용하여 클러스터에 연결해줍니다.

```shell
$ az aks get-credentials --resource-group [리소스 그룹 이름] --name [클러스터 이름]

# 예시
$ az aks get-credentials --resource-group jintube-resource --name jintubeakscluster
```

클러스터가 제대로 연결되었는지 node를 조회해봅니다.

```shell
$ kubectl get nodes
```

## 5. 어플리케이션 실행

어플리케이션을 생성한 Kubernetes 클러스터에 배포해보도록 하겠습니다.

먼저 yaml파일로 Deployment와 Service를 정의해줍니다.

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: jintube-app
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: jintube-app
    spec:
      nodeSelector:
        'beta.kubernetes.io/os': linux
      containers:
        - name: jintube
          image: jintubearc.azurecr.io/jintube:v1
          resources: {}
          env:
            - name: PORT
              value: '4000'
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: jintube-app
  name: jintube-app-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      protocol: TCP
      targetPort: 4000
  selector:
    app: jintube-app
```

위에는 예시입니다.
각자의 어플리케이션 마다 다르게 설정해주셔야 합니다.

`kubectl apply` 명령어로 배포하여줍니다.

```shell
$ kubectl apply -f [파일 이름].yaml

# 예시
$ kubectl apply -f jintube.yaml
```

제대로 배포되었는지 아래의 명령어로 확인해봅니다.

```shell
$ kubectl get service [서비스 이름] --watch

# 예시
$ kubectl get service jintube-app-service --watch
```

위의 명령어를 실행하고 EXTERNAL-IP가 나오면 해당 ip로 접속시 어플리케이션이 작동함을 확인하시면 됩니다.
