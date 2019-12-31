---
title: AWS Lambda에 golang 배포하기 (feat.serverless framework,gin)
date: 2019-04-07 18:00:03
category: development
draft: false
---

해당 예제는 [Serverless Framework](https://serverless.com/)을 사용하여서 aws lambda에 배포합니다.

### 0. Golang 설치

**Mac, Linux**

[Go](https://golang.org/dl/)페이지에서 tar.gz파일을 다운받아서 `/usr/local`에 압축을 풉니다.

```shell
tar -C /usr/local -xzf go1.12.2.src.tar.gz
```

압축을 풀고 난 후에 `~/.profile`에 PATH를 설정합니다.

```shell
sudo vi ~/.profile
```

vi, nano 등으로 해당파일을 불러오고 아래 내용을 추가합니다.

```
export PATH=$PATH:/usr/local/go/bin
```

**Windows**

[Go](https://golang.org/dl/)페이지에서 윈도우에 해당하는 파일(go1.12.2.windows-amd64.msi)을 다운 받아서 실행하셔서 설치하시면 됩니다.

### 1. Serverless Framework 설치

```shell
npm i -g serverless
```

or

```shell
yarn global add serverless
```

위의 명령어로 설치해주시면 됩니다.

### 2. AWS credentials 설정

Serverless Framework을 사용하기 전에 먼저 aws credentials 설정이 필요합니다.
저는 아래 링크를 참고해서 설정하였습니다.
(https://lee-seul.github.io/other/2018/05/13/AWS-Credentials.html)

이후에 `~/.aws/credentials`에서 아래와 같이 설정하시면 됩니다.

```
# ~/.aws/credentials

[lambda]
aws_access_key = xxxxxxxxxxxxx
aws_secret_access_key = xxxxxxxxxxxxxxxxxxxx
```

### 3. IAM 정책 설정

AWS IAM에서 정책을 새로이 json형식으로 추가합니다. (https://gist.github.com/ServerlessBot/7618156b8671840a539f405dea2704c8)

```json
{
  "Statement": [
    {
      "Action": [
        "apigateway:*",
        "cloudformation:CancelUpdateStack",
        "cloudformation:ContinueUpdateRollback",
        "cloudformation:CreateChangeSet",
        "cloudformation:CreateStack",
        "cloudformation:CreateUploadBucket",
        "cloudformation:DeleteStack",
        "cloudformation:Describe*",
        "cloudformation:EstimateTemplateCost",
        "cloudformation:ExecuteChangeSet",
        "cloudformation:Get*",
        "cloudformation:List*",
        "cloudformation:PreviewStackUpdate",
        "cloudformation:UpdateStack",
        "cloudformation:UpdateTerminationProtection",
        "cloudformation:ValidateTemplate",
        "dynamodb:CreateTable",
        "dynamodb:DeleteTable",
        "dynamodb:DescribeTable",
        "ec2:AttachInternetGateway",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:CreateInternetGateway",
        "ec2:CreateNetworkAcl",
        "ec2:CreateNetworkAclEntry",
        "ec2:CreateRouteTable",
        "ec2:CreateSecurityGroup",
        "ec2:CreateSubnet",
        "ec2:CreateTags",
        "ec2:CreateVpc",
        "ec2:DeleteInternetGateway",
        "ec2:DeleteNetworkAcl",
        "ec2:DeleteNetworkAclEntry",
        "ec2:DeleteRouteTable",
        "ec2:DeleteSecurityGroup",
        "ec2:DeleteSubnet",
        "ec2:DeleteVpc",
        "ec2:Describe*",
        "ec2:DetachInternetGateway",
        "ec2:ModifyVpcAttribute",
        "events:DeleteRule",
        "events:DescribeRule",
        "events:ListRuleNamesByTarget",
        "events:ListRules",
        "events:ListTargetsByRule",
        "events:PutRule",
        "events:PutTargets",
        "events:RemoveTargets",
        "iam:CreateRole",
        "iam:DeleteRole",
        "iam:DeleteRolePolicy",
        "iam:GetRole",
        "iam:PassRole",
        "iam:PutRolePolicy",
        "iot:CreateTopicRule",
        "iot:DeleteTopicRule",
        "iot:DisableTopicRule",
        "iot:EnableTopicRule",
        "iot:ReplaceTopicRule",
        "kinesis:CreateStream",
        "kinesis:DeleteStream",
        "kinesis:DescribeStream",
        "lambda:*",
        "logs:CreateLogGroup",
        "logs:DeleteLogGroup",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams",
        "logs:FilterLogEvents",
        "logs:GetLogEvents",
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:DeleteBucketPolicy",
        "s3:DeleteObject",
        "s3:DeleteObjectVersion",
        "s3:GetObject",
        "s3:GetObjectVersion",
        "s3:ListAllMyBuckets",
        "s3:ListBucket",
        "s3:PutBucketNotification",
        "s3:PutBucketPolicy",
        "s3:PutBucketTagging",
        "s3:PutBucketWebsite",
        "s3:PutEncryptionConfiguration",
        "s3:PutObject",
        "sns:CreateTopic",
        "sns:DeleteTopic",
        "sns:GetSubscriptionAttributes",
        "sns:GetTopicAttributes",
        "sns:ListSubscriptions",
        "sns:ListSubscriptionsByTopic",
        "sns:ListTopics",
        "sns:SetSubscriptionAttributes",
        "sns:SetTopicAttributes",
        "sns:Subscribe",
        "sns:Unsubscribe",
        "states:CreateStateMachine",
        "states:DeleteStateMachine"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ],
  "Version": "2012-10-17"
}
```

### 4. 프로젝트 설정

먼저 golang 프로젝트는 GOPATH아래 생성해주셔야 합니다.

`serverless create -t aws-go-dep -p lambda`

위의 명령어로 새로운 lambda에 올라갈 golang프로젝트를 생성합니다.

해당 보일러플레이트는 dep라는 golang의 패키지 매니저와 aws에서 만든 aws-lambda-go라는 패키지를 사용하고 있습니다.

생성하시고 해당 폴더를 열면 hello, world라는 폴더안에 main.go가 들어가고 .gitignore, Gopkg.toml, Makefile, serverless.yml이 있음을 보실 수 있습니다.

먼저 Gopkg.toml 파일은 dep로 설치한 패키지들을 관리할 수 있는 파일입니다.

```toml
[[constraint]]
  name = "github.com/aws/aws-lambda-go"
  version = "1.x"
```

aws-lambda-go라는 라이브러리가 설치되어 있음을 볼 수 있습니다.

다음으로 makefile은 shell 명령을 모아서 실행할 수 있도록 해줍니다.

```makefile
.PHONY: build clean deploy

build:
	dep ensure -v
	env GOOS=linux go build -ldflags="-s -w" -o bin/hello hello/main.go
	env GOOS=linux go build -ldflags="-s -w" -o bin/world world/main.go

clean:
	rm -rf ./bin ./vendor Gopkg.lock

deploy: clean build
	sls deploy --verbose
```

build안에 env로 시작하는 명령어는 람다에서 실행될 함수들이 빌드되도록 해줍니다.
나중에 함수를 바꾸면 이 부분도 변경해주어야합니다.

serverless.yml은 serverless framework의 설정을 해줄 수 있는 파일입니다.

```yml
service: lambda
frameworkVersion: '>=1.28.0 <2.0.0'

provider:
  name: aws
  runtime: go1.x
package:
  exclude:
    - ./**
  include:
    - ./bin/**

functions:
  hello:
    handler: bin/hello
    events:
      - http:
          path: hello
          method: get
  world:
    handler: bin/world
    events:
      - http:
          path: world
          method: get
```

functions 아래에 것들도 위의 makefie의 build처럼 함수를 변경하면 변경해주어야합니다.

배포를 위해 serverless.yml에 아래와 같이 수정해주어야합니다.

```yml
provider:
  name: aws
  runtime: go1.x
  profile: lambda
  region: ap-northeast-2
```

region은 설정해주지 않으면 us-east-1으로 설정됩니다. ap-northeast-2는 서울리전입니다.
profile은 `~/.aws/credentials`에서 설정해준 것을 고르면 됩니다.

이제 배포해보도록 하겠습니다.

```shell
make deploy
```

배포 후에 터미널에 아래 명령어를 실행해줍니다.

```shell
serverless invoke -f world
```

```json
{
  "statusCode": 200,
  "headers": {
    "Content-Type": "application/json",
    "X-MyCompany-Func-Reply": "world-handler"
  },
  "multiValueHeaders": null,
  "body": "{\"message\":\"Okay so your other function also executed successfully!\"}"
}
```

명령어 실행 후 위와 같이 나온다면 성공입니다.

하지만 제가 하고 싶은것은 gin을 사용하는 것이지 aws-lambda-go을 사용하는 것은 아닙니다.

gin을 사용하기위해 gin과 apex에서 만든 gateway를 dep을 통해서 설치해줍니다.

```shell
dep ensure -add github.com/gin-gonic/gin github.com/apex/gateway
```

이제 테스트로 사용한 hello, world를 제거해주고 cmd/handler.go를 생성해줍니다.

```shell
rm -r hello world
mkdir com
touch cmd/handler.go
```

`cmd.handler.go`를 아래와 같이 변경해줍니다.

```go
package main

import (
	"log"
	"net/http"
	"os"

	"github.com/apex/gateway"
	"github.com/gin-gonic/gin"
)

func test(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{"success": true, "message": "test lambda API"})
}

func main() {
	addr := ":3000"
	mode := os.Getenv("GIN_MODE")
	g := gin.New()
	g.GET("/test", test)

	if mode == "release" {
		log.Fatal(gateway.ListenAndServe(addr, g))
	} else {
		log.Fatal(http.ListenAndServe(addr, g))
	}
}
```

만약 gin을 잘 모르신다면 공식문서를 한 번 보고 오는 것을 추천드립니다. (https://gin-gonic.com/)

잘 작동 되는지 로컬에서 확인하도록 하겠습니다.

```shell
go run cmd/handler.go
```

실행하면 아래와 같이 터미널에 표시되게 됩니다.

```shell
[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /test                     --> main.test (1 handlers)
```

제대로 작동되는지 확인해보도록 하겠습니다.

```shell
curl localhost:3000/test
```

```shell
{"message":"test lambda API","success":true}
```

위에서 지정한대로 문구가 잘 나온다면 성공한 것입니다!

이제 로컬에서 제대로 실행되는 것이 확인 되었으니 lambda로 배포해볼 단계가 남은거 같네요.
위의 도입부 쯤에 말씀드린것 처럼 함수를 변경하면 `Makefile`과 `serverless.yml`을 변경해주어야 합니다.

먼저 `Makefile`을 아래와 같이 수정하겠습니다.

```makefile
build:
	dep ensure -v
	env GOOS=linux go build -ldflags="-s -w" -o bin/handler cmd/handler.go
```

다음으로 `serverless.yml`을 아래와 같이 수정히겠습니다.

```yml
functions:
  handler:
    handler: bin/handler
    events:
      - http:
          path: /test
          method: get
    environment:
      GIN_MODE: release
```

이제 배포해보도록 하겠습니다.

```shell
make deploy
```

배포 후 나오는 `ServiceEndpoint`에 `/test`를 붙여서 테스트해보도록 하겠습니다.

```shell
url https://blabla.execute-api.ap-northeast-2.amazonaws.com/dev/test
```

```shell
{"message":"test lambda API","success":true}⏎
```

로컬에서 사용한 것과 같이 동일하게 나옵니다.

api의 경우 더 여러경우가 많은데 그 것에 대해서는 아래의 서버리스 플랫폼의 문서를 보시면 대부분이 해결됩니다.
https://serverless.com/framework/docs/providers/aws/events/apigateway/

위의 실행해본 소스는 제 깃허브에 남겨두었습니다.
https://github.com/seongjoojin/lambda-go-gin

### 5. 마치며

golang에는 정말 많은 웹프레임워크가 있어서 고르기 어려웠던거 같습니다.

그래서 대충 정리해두려고 합니다.

- 풀스택 프레임워크 : [Revel](https://revel.github.io/), [Beego](https://beego.me/)
- 마이크로 프레임워크 : [Gin](https://gin-gonic.com/), [Echo](https://echo.labstack.com/), [Martini](https://github.com/go-martini/martini)
- 라이브러리/툴킷
  - 라우터 : https://github.com/gorilla/mux, https://github.com/julienschmidt/httprouter, https://github.com/bmizerany/pat
  - 컨텍스트 : https://github.com/gorilla/context, https://godoc.org/golang.org/x/net/context
  - 미들웨어 : https://github.com/justinas/alice
  - 렌더러 : https://github.com/unrolled/render
- ORM : [gorm](http://gorm.io/), [xorm](http://xorm.io/docs)
- log : [logrus](https://github.com/sirupsen/logrus)
