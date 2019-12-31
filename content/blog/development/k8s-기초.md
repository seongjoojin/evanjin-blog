---
title: k8s 기초
date: 2019-12-17 10:12:21
category: development
draft: false
---

본 내용은 eks workshop의 내용을 정리한 것입니다.

## k8s node

- 쿠버네티스 클러스터를 구성하는 머신들
- 마스터 노드(Master-node) : Control Plane을 형성하고 클러스터의 두뇌 역할
- 워커 노드(Worker-node) : Data Plane을 형성하고 파드(pod)를 통해 실제 컨테이너 이미지를 작동 시팀

## k8s 오브젝트

- 쿠버네티스의 오브젝트(objects)는 클러스터의 상태를 나타내는 단위(entities)
- 쿠버네티스는 항상 오브젝트의 현재 상태를 의도한 상태(desired state)와 동일하게 만들게끔 작동함

의도한 상태란?

- 어떤 파드(컨테이너)들이 어느 노드에서 동작(running) 중인지
- 동작 중인 컨테이너 레플리카(replicas)의 개수

## k8s 오브젝트 세부내용

- 파드(Pod) : 하나 이상의 컨테이너를 둘러싼 가장 작은 래퍼(wrapper) 단위
- 데몬셋(DaemonSet) : 워커 노드에 파드의 단일 인스턴스를 실행함
- 디플로이먼트(Deployment) : 어플리케이션 버전의 롤아웃(또는 롤백) 방법에 대한 세부내용
- 레플리카 셋(ReplicaSet) : 계속 동작할 파드의 개수를 정의함
- 잡(Job) : 파드가 제대로 완성되어 동작할 수 있도록 함
- 서비스(Service) : 고정 IP 주소를 파드의 논리 그룹과 매핑함
- 레이블(Label) : 연결과 필터링에 사용되는 키/밸류 쌍
