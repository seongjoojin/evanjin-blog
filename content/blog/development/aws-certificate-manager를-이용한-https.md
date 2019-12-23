---
title: AWS Certificate Manager를 이용한 HTTPS
date: 2019-12-23 17:12:84
category: development
---

## 0. 도메인 구입

NameCheap, aws router 53, name.com, cafe24 등에서 구매 가능

## 1. 인증서 등록

AWS Certificate Manager에서 위에서 구매한 도메인 연결

Tag 참고 url
https://docs.aws.amazon.com/acm/latest/userguide/tags.html

5단계에서 나오는 CSV파일에 있는 CNAME Record의 Record Name, Record Value를 도메인 구입한 곳에서 등록해줍니다.
