---
title: react native 0.63.0 변화된 점
date: 2020-07-09 22:07:38
category: development
draft: true
---

한국 시간으로 새벽 3시쯤 react native의 0.63.0 버전이 릴리즈 되었습니다.

0.63.0의 변경사항을 알아보도록 하겠습니다.

## 변경사항

변경사항으로는 첫번째로는 LogBox라는 디버그 모드에서 오류 날 때 생기는 로그가 기본으로 적용되게 되었습니다.

![logbox.png](https://reactnative.dev/blog/assets/0.63-logbox.png)

0.62.0에서는 옵션으로 존재하였으나 0.63.0에서는 기본으로 적용되게 되었습니다.
기존의 로그에 비하여서 훨씬 보기 쉬워졌고 조금 더 이뻐졌습니다.

두번째 변경사항으로는 Touchable 시리즈들을 대체할 Pressable 컴포넌트가 생겼습니다.

세번째로는 Flutter의 Colors Class 에 자극받은 것인지 Native Colors라는 것을 추가하였습니다.

네번째로는 iOS 9 과 Node.js 8 지원을 더이상 하지 않도록 변경되었습니다.

기타 개선 사항으로는 View, Text 컴포넌트에 width, hegiht를 주지 않더라도 렌더링 될 수 있도록 지원하기 시작하였고
iOS LaunchScreen 라는 것이 xib에서 storyboard로 변경되었다고 합니다.

https://github.com/react-native-community/releases/blob/master/CHANGELOG.md#v0630
