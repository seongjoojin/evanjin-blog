---
title: AWS EKS에 Node App 배포해포기
date: 2019-12-20 12:12:72
category: development
---

## 0. Intro

이번 포스팅은 heroku로 배포한 애플리케이션을 AWS EKS로 마이그레이션한 삽질기를 공유해보려고 합니다.

먼저 저는 노마드코더를 통해서 만든 유튜브 클론을 heroku에 배포해놓은 상태였습니다.

공짜로 쓰기에는 heroku도 나쁘진 않았지만 콜드 스타드할 때는 너무 느린 것이 단점이였습니다.

대체할 것을 아는 분께 여쭤보다가 AWS Fargate로 가는 것도 방법이라고 안내해주셔서 삽질을 해보려고 합니다.

## 1. AWS EKS

AWS EKS는 무엇일까요?

AWS EKS는 관리형 Kubernetes 서비스입니다.

사실 AWS ECS를 사용할 수 도 있지만 Kubernetes로 시도해보고 싶어서 AWS EKS를 선택하였습니다.

관리형이란 AWS EB와 같이 사용자가 직접 모든 것 조작하지않고 AWS에서 관리해주고

## 3. Dockerizing

### 3-1. Dockerfile 생성

먼저 제가 만든 NodeJS 어플리케이션의 `Dockerfile`을 생성해줍니다.

```Dockerfile
FROM node:lts

WORKDIR /app
ADD . /app/

RUN npm i -g npm && npm i -g pm2

RUN rm package-lock.json
RUN npm i && npm run build

ENV HOST 0.0.0.0
EXPOSE 4000

CMD ["npm", "run", "start"]
```

Ddockerfile은 해당 NodeJS 어플리케이션마다 다르기 때문에 구성하신 scripts나 설정마다 다르게 해주어야합니다.

docker에서의 pm2 설정 같은 경우에는 pm2의 공식 홈페이지에 더욱 자세히 나와있으므로 참고하시길 바랍니다.

https://pm2.keymetrics.io/docs/usage/docker-pm2-nodejs/

Dockerfile을 만드셨다면 `.dockerignore` 파일도 생성해줍니다.

```
node_modules
```

저는 node_modules만 넣었지만 컨테이너에 포함되지 말아야할 것들이 있다면 추가해주시면 됩니다.

### 3-2. Docker image 생성 및 테스트

다음으로 Docker 컨테이너 이미지를 생성해줍니다.

```shell
docker build -t jintube .
```

여기서 jintube 부분이 이미지의 이름이 되게 되는데요.
이부분은 여러분들이 하시고 싶은 이름으로 진행하셔야 합니다.

이 후에 테스트로 이 Docker 이미지를 실행해보도록 하겠습니다.

```shell
docker run -it -p 4000:4000 jintube
```

포트와 이미지 이름은 생성하신 것에 따라 다를 수 있으니 조정해서 사용하시면 됩니다.

## 4. AWS ECR

AWS ECR에 위에서 생성한 Docker 컨테이너 이미지를 올리는 과정을 진행합니다.

AWS ECR이란 Docker 컨테이너 이미지를 손쉽게 저장, 관리 및 배포할 수 있게 해주는 Docker 컨테이너의 저장소입니다.
