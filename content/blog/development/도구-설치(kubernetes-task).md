---
title: 도구 설치(kubernetes task)
date: 2019-12-12 17:02:03
category: development
---

## 0. Intro

본 블로그글은 k8s task들을 따라하면서 정리한 글입니다.
실습해보고 정리를 안 하면 잊어버려서 잊어버리지 않도록 정리해봅니다.
참고로 저는 mac을 사용하고 있어서 모든 실습 정리는 mac이 기준입니다.

## 1. kubectl 설치

맥에는 homebrew라는 편리한 패키지 매니저가 있습니다.
저는 homebrew를 통해서 설치하도록 하겠습니다.

```shell
brew install kubectl
```

or

```shell
brew install kubernetes-cli
```

설치 후 제대로 설치가 되었는지 kubectl 버전을 확인해봅니다.

```shell
kubectl version
```

## 2. Minikube 설치

**가상화 지원여부 확인**

먼저 맥에서 가상화 지원여부를 확인합니다.

```shell
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```

출력 중에서 VMX를 볼 수 있다면 가상화 지원을 하는 것입니다.

**하이퍼바이저 설치**

먼저 minikube를 설치하기 전에 하이퍼 바이저를 설치해줍니다.
저는 virtualbox를 설치하려고 합니다.

```shell
brew cask install virtualbox
```

설치 후 virtualbox가 제대로 동작된다면 제대로 설치된 것입니다.

**minikube 설치**

```shell
brew install minikube
```

설치 후 아래 명령어가 제대로 실행되는지 확인해봅니다.

```shell
minikube version
```

여기까지 성공하셨다면 필요한 도구는 설치를 완료하였습니다.
다음으로는 클러스터 운영에 대해서 실습하면서 정리해보도록 하겠습니다.
