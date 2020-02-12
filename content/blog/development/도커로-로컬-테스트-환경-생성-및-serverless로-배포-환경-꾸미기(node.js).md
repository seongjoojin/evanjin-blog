---
title: 도커로 로컬 테스트 환경 생성 및 serverless로 배포 환경 꾸미기(node.js)
date: 2020-02-07 15:02:09
category: development
draft: false
---

### 0. 감사인사

먼저 서버리스에 대해서 모르는 것들에 대해서 많은 조언을 주신 노마드코더의 플린님과 대디님께 감사를 드립니다.

마커스님께서도 좋은 코드를 짜주셔서 많은 도움이 되었습니다. 감사합니다.

### 1. 로컬 테스트 환경 생성 (도커)

먼저 제가 테스트한 환경은 Mac os 입니다.
윈도우, 리눅스 사용하시는 분들은 일정 부분이 다를 수 있습니다.

사실상 프론트의 로컬테스트 같은 경우는 특별히 도커로 구성 안 해도 되지만

백엔드의 경우 디비 설정과 서버 설정을 같이 해야하는 경우가 많아서
처음 설정에서 많은 삽질을 할 가능성이 농후합니다.

제가 해커톤에서 사용하였던 node.js 백엔드를 예로 들어보겠습니다.

#### 1-1. dockerignore 파일 생성 및 설정

먼저 `.dockerignore` 파일을 생성해줍니다.

이 파일의 역할을 docker의 이미지를 만들때 필요없는 폴더 및 파일이 들어가지 않도록 해주는 역할을 해줍니다.

```text
node_modeuls
```

저는 node_modeuls를 넣어주었는데 이유는 docker의 이미지 크키가 쓸데없이 커질거 같고 라이브러리들이 버전업할때마다 달라질 것이기때문에 넣어주었습니다.

#### 1-2. Dockerfile 생성 및 설정

```Dockerfile
FROM node:lts

WORKDIR /usr/app
COPY package*.json ./

RUN npm i
COPY . .

EXPOSE 8888

CMD ["npm","run","dev"]
```

혹시나 Dockerfile의 구성에 대해서 모르실 분들(미래의 나)을 위해서 위의 내용을 하나하나 살펴보도록 하겠습니다.

`FROM node:lts`
도커 허브라는 도커의 공개 레파지토리에서 node의 lts 버전의 이미지를 받아옵니다

`WORKDIR /usr/app`
WORKDIR는 RUN, CMD 등의 명령이 실행될 디렉토리를 정해주는 설정입니다. 저는 `/usr/app`으로 설정하였습니다.

`COPY package*.json ./`
라이브러리 설치를 위해 package.json 등을 복사합니다.
`RUN npm i`
npm을 실행하여 라이브러리들을 설치합니다.

`COPY . .`
이제 모든 폴더와 파일을 복사합니다.

`EXPOSE 8888`
EXPOSE는 특정 포트를 열어줄때 사용하는 설정입니다. (포트포워드와 비슷하다고 보시면 됩니다.)
저 같은 경우는 8888포트를 사용하고 있어서 해당 포트를 열어주는 것으로 설정하였습니다.

`CMD ["npm","run","dev"]`
CMD는 명칭 그대로 명령어를 실행해주면 저 같은 경우는 `npm run dev`가 실행되도록 설정해주었습니다.

이외에는 많은 설정들이 존재하는데요.
그 부분은 공식문서를 통해서 확인해주시면 됩니다.

https://docs.docker.com/engine/reference/builder/

다음으로 docker-compose를 설정해줍니다.

docker-compose는 하나의 설정파일로 여러개의 컨테이너를 정의하고 실행할 수 있는 기능입니다.

사실상 위의 Dockerfile이랑 mysql 이미지가지고 따로 docker를 두개 돌려도 충분하지만 귀찮음이 클거 같음으로 docker-compose로 진행하였습니다.

#### 1-3. docker-compose 설정

```yaml
version: '3.7'

networks:
  app-tier:
    driver: bridge
services:
  mysql:
    networks:
      - app-tier
    image: mysql:5.7
    restart: always
    ports:
      - '3306:3306'
    environment:
      - MYSQL_ROOT_PASSWORD=anyBookLog
      - MYSQL_DATABASE=anyBookLog
      - MYSQL_USER=anyBookLog
      - MYSQL_PASSWORD=anyBookLog
    command:
      [
        '--character-set-server=utf8mb4',
        '--collation-server=utf8mb4_unicode_ci',
      ]

  api:
    networks:
      - app-tier
    build:
      context: .
      dockerfile: ./Dockerfile
    image: anybooklog-api:0.1.0
    container_name: anybooklog-api
    environment:
      - PORT=8888
    volumes:
      - ./src:/usr/app/src
    env_file: ./src/.env
    restart: always
    ports:
      - '8888:8888'
    depends_on:
      - mysql
    links:
      - mysql
```

혹시나 docker-compose를 모를 미래의 저를 위해서 간략하게 정리해보도록 하겠습니다.

`version: '3.7'`
버전은 사실상 도커 엔진 버전에 따라서 호환되는 것이 다른데요.
저 같은 경우는 로컬의 도커는 최신 버전이기 때문에 최근 버전인 3.7로 명시하였습니다.

```yaml
networks:
  app-tier:
    driver: bridge
```

여기서 특별하게 app-tier라는 이름의 network를 지정해주고 bridge로 설정해주었습니다.

사실상 저도 네트워크를 잘 모르기 때문에 맞는 설명인지 모르겠지만
간단하게 다중 컨테이너 간의 분리된 네트워크 정도로 이해하고 사용하였습니다.

다음으로 services를 설정해주는 부분인데요.
services는 사실상 컨테이너를 설정해주는 부분 이라고 보시면 간단합니다.

너무 길기 때문에 핵심적인 것들만 살펴보도록 하겠습니다.

```yaml
api:
  networks:
    - app-tier
```

먼저 위에서 설정한 app-tier로 네트워크를 구성합니다.

```yaml
image: mysql:5.7
```

image는 컨테이너에 사용될 이미지들을 설정해줍니다.

```yaml
build:
  context: .
  dockerfile: ./Dockerfile
image: anybooklog-api:0.1.0
```

app에서는 mysql과 달리 image를 지정하기 전에 build되도록 설정해주었습니다.

```yaml
environment:
  - MYSQL_ROOT_PASSWORD=anyBookLog
  - MYSQL_DATABASE=anyBookLog
  - MYSQL_USER=anyBookLog
  - MYSQL_PASSWORD=anyBookLog
```

```yaml
env_file: ./src/.env
```

docker-compose에서 환경변수를 정해주는 방법이 제가 아는 한도 내로는 위의 두가지로 알고 있습니다.

environment에서 직접 정해주거나 .env 파일 등으로 설정해주는 것입니다.
만약 보안 등에 치명적인 이미지라면 env_file 설정을 쓰는 것이 좋을거 같고
아니라면 environment에 직접 지정해주어도 문제는 없을 것으로 생각 됩니다.

```yaml
volumes:
  - ./src:/usr/app/src
```

api에서 volumes를 위와 같이 정의한 것은 src안의 파일 내용을 변경시(소스 코드를 변경시)
nodemon 등으로 핫로딩하여 바로 결과를 볼 수 있도록 설정한 것이고
좀더 자세히 설명하자면 Docekrfile에서 정한 WORKDIR이 현재 폴더의 src를 가르키도록 설정한 것입니다.

```yaml
command:
  ['--character-set-server=utf8mb4', '--collation-server=utf8mb4_unicode_ci']
```

command는 도커 내부에서 해당 명령이 실행되고 컨테이너 만들어지도록 합니다.
미세먼지 팁이지만 위의 명령어는 mysql에서 이모지를 사용할 수 있도록 설정해주는 명령어입니다.

```yaml
depends_on:
      - mysql
    links:
      - mysql
```

마지막으로 중요한 depends_on과 links입니다.

depends_on의 경우는 지정한 서비스가 실행된 뒤에 서비스가 실행되겠다는 것으로
위의 예를 들면 mysql 이라는 이름의 서비스 실행이 되고나서 app 이라는 이름의 서비스가 실행된다는 것입니다.

links는 해당 컨테이너에서 지정한 컨테이너 서비스를 명시한 이름으로 참조할 수 있음을 설정해주는 옵션입니다.
위의 예를 들면 mysql 이라는 이름의 서비스를 app 서비스에서 mysql로 불러서 참조할 수 있게 된다는 것입니다.

이제 이렇게 설정을 다 해주셨으면 아래의 명령으로 로컬에서 테스트할 수 있게 됩니다.

장고가 예제이긴 하나 아래 링크의 글도 한 번 읽어보시길 추천드립니다.

https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose

```bash
$ docker-compose up --build
```

로컬 테스트 환경이라 -d 옵션은 빼주었지만 혹시나 백그라운드에서 돌게 하고 싶으시면 아래와 같이 명령어를 바꿔주시면 됩니다.

```bash
$ docker-compose up -d --build
```

### 2. serverless 배포 설정하기

#### 2-1. 설치 및 초기 설정

먼저 servlerss 를 설치하여 줍니다.

```bash
$ curl -o- -L https://slss.io/install | bash
```

다음으로 aws에서 설정해주어야 하는 것들이 있는데요.

먼저 아래의 링크의 안내대로 IAM를 설정해줍니다.

https://serverless-stack.com/chapters/ko/create-an-iam-user.html

생성 후에 받은 Access key ID와 Secret access key를 통해 serverless를 설정해줍니다.

```bash
$ serverless config credentials --provider aws --key [Access key ID] --secret [Secret access key]
```

#### 2-2. 프로젝트에 serverless 라이브러리 설치

```bash
yarn add -D serverless serverless-http serverless-dotenv-plugin serverless-offline serverless-prune-plugin
```

몇가지 라이브러리들만 설명하자면 아래와 같습니다.

- serverless-offline 라이브러리는 로컬에서 서버리스를 실행하여 디버그 할 수 있게 도와줍니다.
- serverless-prune-plugin는 서버리스가 배포할 때 AWS S3를 사용하여 서버리스 코드가 버저닝 되나 리전 당 용량 제한이 있으므로(75기가) 이전 버전들의 갯수를 제한 해주는 라이브러리입니다. (prune이라는 flag이 있으면 캐쉬되었던 artifact들 청소한다고 보면 된다고 지인이 첨언해주셨습니다.)

#### 2-3. serverless.yaml 파일 설정

```yaml
service: anybooklog-api

plugins:
  - serverless-offline
  - serverless-dotenv-plugin
  - serverless-prune-plugin

custom:
  defaultStage: pord
  currentStage: ${opt:stage, self:custom.defaultStage}
  prune:
    automatic: true
    number: 3
  dotenv:
    path: src/.env

provider:
  name: aws
  runtime: nodejs12.x
  region: ap-northeast-2
  stage: ${self:custom.currentStage}
  deploymentBucket:
    name: anybooklog-api
  memorySize: 256
  timeout: 3
  logRetentionInDays: 3
  environment:
    TZ: Asia/Seoul
    NODE_ENV: production

functions:
  app:
    handler: src/serverless.handler
    events:
      - http:
          path: /
          method: ANY
          cors: true
      - http:
          path: /{any+}
          method: ANY
          cors: true
```

분명히 잊어버릴 미래의 저를 위해서 중요한 것들을 하나씩 살펴보도록 하겠습니다.

```yaml
plugins:
  - serverless-offline
  - serverless-dotenv-plugin
  - serverless-prune-plugin
```

plugins에 사용할 플러그인들을 지정해줍니다.

```yaml
custom:
  defaultStage: pord
  currentStage: ${opt:stage, self:custom.defaultStage}
  prune:
    automatic: true
    number: 3
  dotenv:
    path: src/.env
```

custom에서는 Stage이나 플러그인 등의 설정을 해줍니다.

플러그인 들을 각각의 깃허브 페이지마다 custom에서 어떻게 설정해주는지 명세되어 있으니 그 부분을 참고하시면 됩니다.

```yaml
provider:
  name: aws
  runtime: nodejs12.x
  region: ap-northeast-2
  stage: ${self:custom.currentStage}
  deploymentBucket:
    name: anybooklog-api
  memorySize: 256
  timeout: 3
  logRetentionInDays: 3
  environment:
    TZ: Asia/Seoul
    NODE_ENV: production
```

다음으로 provider입니다.
provider는 클라우드 공급자와 설정 등을 지정하여줍니다.
이름, 런타임, 리전 등등을 설정해줍니다.

참고로 environment에 설정해야 serverless로 올라간 어플리케이션도 해당 환경 변수를 읽을 수 있습니다.
저는 이걸 몰라서 `process.env.NODE_ENV="production"`해주고 왜 안 되는지 삽질 했습니다.

```yaml
functions:
  app:
    handler: src/serverless.handler
    events:
      - http:
          path: /
          method: ANY
          cors: true
      - http:
          path: /{any+}
          method: ANY
          cors: true
```

마지막으로 functions를 설정해줍니다.
사실상 람다에서는 function에 정리되는 것들이 express로 보면 router라고 보시면 편할거 같습니다.

위에서는 핸들러에서 express에서 설정한 app을 리턴하도록 설정해줘서 어떤 path든 express에서 설정한 router가 정한대로 작동하도록 해줍니다.

#### 2-4. serverless.yaml 파일 설정대로 파일 생성 및 프로젝트 설정

저 같은 경우 handler를 `src/serverless.handler` 로 지정하였으므로 src폴더 아래 serverless.js 파일을 생성하여 아래와 같이 설정해주었습니다.

`src/serverless.js`

```js
const serverless = require('serverless-http')
const app = require('./app')

module.exports.handler = serverless(app)
```

같은 폴더 안의 app.js에서는 express를 설정해준 후 module로 export 하였습니다.

`src/app.js`

```js
const express = require('express')
const morgan = require('morgan')
const cors = require('cors')
const cookieParser = require('cookie-parser')
const dotenv = require('dotenv')
const passport = require('passport')
const hpp = require('hpp')
const helmet = require('helmet')

const db = require('./models')
const authAPIRouter = require('./routes/auth')
const userAPIRouter = require('./routes/user')

dotenv.config()

const app = express()
db.sequelize.sync()
const prod = process.env.PRODUCTION

if (prod) {
  app.use(helmet())
  app.use(hpp())
  app.use(mogran('combined'))
} else {
  app.use(morgan('dev'))
}
app.use(cors())

app.use(express.json())
app.use(express.urlencoded({ extended: true }))
app.use(cookieParser(process.env.COOKIE_SECRET))
app.use(passport.initialize())

app.get('/', (req, res) => {
  res.send('any book log 백엔드 정상 동작!')
})

app.use('/api/auth', authAPIRouter)
app.use('/api/user', userAPIRouter)

module.exports = app
```

마지막으로 도커를 이용한 로컬 테스트(nodemon)에도 사용해야 하므로 따로 원래의 express 서버 설정처럼 포트와 listening 설정을 따로 server.js로 분리하였습니다.

`src/server.js`

```js
const app = require('./app')

const PORT = process.env.PORT || 8888

const handleListening = () =>
  console.log(`✅  Listening on: http://localhost:${PORT}`)

app.listen(PORT, handleListening)
```

#### 2-5. script 설정 및 배포

`package.json`

```json
"scripts": {
  "dev:server": "nodemon --delay 2",
  "dev:serverless": "sls offline start",
  "deploy": "sls deploy"
}
```

dev:server는 기존의 로컬테스트와 도커 테스트를 위하여 분리하였고
dev:serverless는 serverless를 로컬에서 테스트하여서 배포시에 문제가 없도록 체크할 수 있도록 하였습니다.

deploy로 배포되도록 분리하였는데 리전이나 stage 등을 다르게주셔서 분리해도 괜찮을듯 싶습니다.

https://serverless.com/framework/docs/providers/aws/cli-reference/deploy/

혹시나 node.js 앱을 배포시 bcrypt 때문에 문제가 발생하신다면 bcryptjs를 설치해주시면 됩니다!

### 3. 작은 팁들

쓸모 없을 수 도 있지만 환경을 꾸미면서 개인적으로는 유용한게 사용했던 팁을 정리해봅니다.

리눅스/맥 랜덤한 문자열 넣기

```bash
$ openssl rand -hex 64
```

도커 컨테이너 내부 접근

```bash
# 도커 컨테이너 조회
$ docker ps
# 도커 내부 접근
$ docker exec -it [컨테이너 아이디] /bin/bash
```

맥에서 hyperkit 죽이기

```bash
# hyperkit이 어느 PID를 쓰는지 조회
$ ps -ef | grep hyperkit
# 프로세스 죽이기
$ sudo kill -9 [PID]
```

serverless 프레임워크에서 권장하는 aws iam 설정

https://gist.github.com/ServerlessBot/7618156b8671840a539f405dea2704c8

부족한 글 읽어주셔서 감사합니다.
