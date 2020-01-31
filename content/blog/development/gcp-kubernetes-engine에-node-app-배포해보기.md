---
title: GCP kubernetes engine에 Node App 배포해보기
date: 2019-12-22 23:12:99
category: development
draft: false
---

## 0. Intro

이번 포스팅은 heroku로 배포한 애플리케이션을 GCP kubernetes engine으로 마이그레이션한 삽질기를 공유해보려고 합니다.

먼저 제가 만든 유튜브 클론을 heroku에 배포해놓은 상태였습니다.

요즘에 쿠버네티스를 공부했지만 실제 프로덕트에 올려서 해본 적은 없는거 같아서 해당 클론을 한 번 kubernetes에 올려서 실제 어플리케이션을 돌려보고 싶었습니다.

## 1. GCP kubernetes engine란

AWS EKS와 Azure AKS와 같이 완전 관리형 서비스라고 볼 수 있습니다.
처음에 무료 크레딧을 주는 GCP와 Azure 중에 고민하다가
구글링 했을때 자료가 GCP쪽이 더 많았기 때문에 GCP쪽을 선택하게 되었습니다.
시간이 된다면 다른 어플리케이션은 Azure쪽으로 해볼 예정입니다.

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

## 3. GKE 클러스터 만들기

### 3-1. google clould sdk 설치

https://cloud.google.com/sdk/install

위의 링크를 들어가셔서 os에 맞는 설치법대로 설치하시면 됩니다.

저는 mac을 사용하고 있으므로 mac만 안내드리자면 아래와 같습니다.

```shell
brew cask install google-cloud-sdk
```

homebrew로 간단하게 설치해줍니다

```shell
gcloud auth login
```

로그인을 해줍니다.

### 3-2. gcloud 기본 설정

먼저 기본 프로젝트를 설정해줍니다.

```shell
gcloud config set project [project-id]
```

다음으로 리전을 선택해줍니다.
포스팅을 하는 지금은 서울이 생기지 않았지만 곧 생길 것으로 보입니다.
저는 일단 가장 가까운 도쿄로 하려고 합니다.

```shell
gcloud config set compute/zone asia-northeast1-a
```

### 3-3. 클러스터 생성

이제 드디어 클러스터를 생성합니다.

```shell
gcloud container clusters create [cluster-name]
```

만드는데 몇 분 정도 소요될 수 있습니다.

기본적으로 노드가 3개 만들어지는데요.
이것이 맘에 들지 않으시면 직접 Kubernetes Engine 페이지에서 생성하셔도 됩니다.

만들어진 후에는 사용자 인증 정보를 얻어냅니다.

```shell
gcloud container clusters get-credentials [cluster-name]
```

## 4. 클러스터에 어플리케이션 배포하기

### 4-1. Container Registry에 도커 이미지 추가하기

먼저 Container Registry에 대한 요청을 인증하도록 Docker를 구성되도록 아래 명령어를 실행합니다.

```shell
gcloud auth configure-docker
```

다음으로 레지스트리 이름으로 이미지에 태그 지정을 해줍니다.

```shell
docker tag [image name] gcr.io/[PROJECT-ID]/[image name]:[tag name]

# 예시
docker tag jintube:0.1.0 gcr.io/jintube/jintube-image:0.1.0
```

마지막으로 이미지를 Container Registry로 추가해줍니다.

```shell
docker push gcr.io/[PROJECT-ID]/[image name]:[tag name]

# 예시
docker push gcr.io/jintube/jintube-image:0.1.0
```

### 4-2. 배포하기

먼저 배포를 생성하여 줍니다.

```shell
kubectl create deployment [name]--image=[image name]

# 예시
kubectl create deployment jintube --image=gcr.io/jintube/jintube-image:0.1.0
```

다음으로 어플리케이션을 배포한 후에 사용자가 엑세스할 수 있도록 인터넷에 노출해주어야 합니다.

```shell
kubectl expose deployment [name] --type LoadBalancer \
  --port [port] --target-port [target-port]

# 예시
kubectl expose deployment jintube --type LoadBalancer \
  --port 80 --target-port 4000
```

마지막으로 배포된 어플리케이션이 제대로 동작되는지 확인해봅니다.

```shell
kubectl get pods
```

실행 중인 Pod를 먼저 검사합니다.

```shell
kubectl get service [service name]

# 예시
kubectl get service jintube
```

여기서 service name은 위에서 kubectl create deployment 명령어로 만든 이름입니다.
위의 명령어를 실행 후 EXTERNAL-IP가 나오게 되는데요.
해당 ip를 브라우저로 접속 후 정상적으로 작동하는지 확인합니다.
