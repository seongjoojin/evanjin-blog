---
title: Vue 개발 시 참고사항
date: 2019-05-04 23:01:03
category: development
---

## 0. app의 특징에 따른 Vue 플러그인 조합

**Vue Router와 Vuex 모두 사용할 필요 없는 app의 예**

- 기존 전자상거래 사이트처럼 라우팅은 서버 사이드에서 수행하고 클라이언트 사이드도 컴포넌트로 구성되지않는 app
- 서비스 소개 사이트 등 일부 컴포넌트가 동적으로 동작하는 랜딩페이지

**Vue Router만 적용하면 되는 app의 예**

- SPA 기반 관리 화면 등 각 페이지에서 간단한 기능을 제공하는 app
- native app처럼 경쾌하게 동작하는 클라이언트 페이지 이동을 제공하는 app 혹은 game

**Vuex만 적용하면 되는 app의 예**

- 대시보드, 채팅 app, 사진 가공 app 등 한 페이지 안에서 여러 컴포넌트 간의 데이터 연동이 필요한 단일 도구 형태의 app

**Vue Router와 Vuex를 모두 적용해야 하는 app의 예**

- 메일 클라이언트나 캘런더 클라이언트 등 여러 페이지에서 복잡한 컴포넌트 구성이 예상되는 대규모 SPA

## 1. Vue CLI의 설정

e2e test를 위해서 자바 jdk설치 필요

- ESLint: Standard
- unit test: yes
- test runner: mocha (다른 유닛 테스트 툴 선택도 상관없음)
- e2e test (nightwatch): yes

## 2. vue cli 단위 테스트 및 e2e test 툴

https://cli.vuejs.org/config/#unit-testing

Jest: React진영에서 많이 사용되는 유닛테스트 툴로 Vue진영에서도 많이 사용하는 편
Mocha: 어설션 검증 실행 환경을 제공하는 테스트 라이브러리

https://cli.vuejs.org/config/#e2e-testing

Cypress: Vue conf에서 발표된 것에서 처음 본 e2e test 툴이였고 꾸준히 발전 중으로 보임
Nightwatch: 셀레니움 기반으로 돌아가며 전부터 vue진영에서 꽤나 쓰인 것으로 보임

## 3. 아토믹 디자인

http://bradfrost.com/blog/post/atomic-web-design/

https://dev.to/miladalizadeh/vue-cli-30-plugin-for-creating-apps-using-atomic-design--storybook-42dk

**원자**

원자는 UI를 구성하는 기본 요소이며, 기능적으로 더 이상 분할할 수 없는 대상.

ex) 텍스트박스, 체크박스, 버튼, 아이콘, 레이블 등

**분자**

원자를 조합해 만든 요소

ex) 로그인 폼, 페이지 네비게이션 등

**유기체**

유기체는 원자, 분자, 유기체가 모여 구성되는 요소

ex) 채팅 목록, 칸반 목록 등

**템플릿**

템플릿은 분자와 유기체가 조합되어 만들어진 페이지 템블릿. 와이어프레임

ex) 로그인 뷰 등

## 4. Vue DevTools 사용하기

https://github.com/vuejs/vue-devtools

크롬 확장 프로그램 : https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd
파이어폭스 확장 프로그램 : https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/
일렉트론 앱: https://github.com/vuejs/vue-devtools/blob/master/shells/electron/README.md

## 5. 성능 측정

**성능 측정 설정 방법**

Vue.config.preformance를 true로 설정 후 `npm run dev` 명령을 실행해서 개발 서버를 실행한 다음, 구글 크롬 개발자도구에서 preformance 탭을 선택하고 애플리케이션을 새로고침하면 성능이 측정됩니다.

Vue.js에서 자바스크립트 실행에 소요된 시간이 시간 순서로 시각화 되어 나타나게 됩니다.

render는 라이브러리 사용자(애플리케이션 개발자)가 사용할 일이 많으므로 그만큼 개선의 여지가 많습니다.

**렌더링 성능 개선**

중복 처리가 일어나는 곳은 없는지 확인해 개선하는 방법이 정석입니다.

추가적으로 다음과 같은 방법으로 렌더링 성능을 개선할 수 있습니다.

- `v-if`와 `v-show`를 구분해서 사용 (평가 값이 빈번하게 변하는 바뀌는 경우에는 `v-show`를 사용, 평가 값이 자주 변하지 않는 경우에는 `v-if`가 적합) => 스타일을 수정하는 쪽보다 DOM을 수정하는 쪽이 렌더링 비용이 더 큼
- 데이터바인딩은 메서드보다는 계산 프로퍼티를 사용 => 무조건 좋은 것은 아님 (실시간 데이터 대응 등에는 불리)
- 계산 프로퍼티와 와처를 구분해서 사용 => 반복문처럼 연산비용이 높거나 비동기 처리가 따르는 경우에는 계산 프로퍼피 대신 와처를 사용하는 것이 나음
- v-for를 사용해 리스트를 렌더링할 때는 되도록 key 속성을 사용 => Vue의 가상돔의 diff/patch에서 key 속성으로 연결된 DOM 요소를 재사용하므로 DOM 조작 연산 비용이 최소한으로 억제됨
- v-once로 컴포넌트 콘텐츠를 캐싱 => v-once는 렌더링 대상 콘텐츠의 값을 처음 한 번만 평가하므로 두 번째부터는 캐시된 값을 사용함. (요소는 매번 렌더링하고 싶지않는 경우 유용함)
- 함수형 컴포넌트를 사용 => 컴포넌트 내부의 상태 없이 프로퍼티만으로 컴포넌트를 렌더링하는 경우의 성능을 개선하는데 활용할 수 있음.
- 템플릿을 미리 컴파일하기
- 템플릿 컴파일러 옵션을 사용

## 6. 개발 툴

**Storybook**

https://storybook.js.org

UI 개발 환경을 제공하는 도구입니다.

Storybook을 사용하면 다음과 같은 장점이 있습니다.

- 컴포넌트의 input 요소와 button 요소등이 사용자와 상호 작용 확인
- 컴포넌트의 CSS, 글꼴, 이미지 등 UI의 외관 확인
- 컴포넌트의 목록을 만들어 프로젝트에서 사용되는 현황 파악
- 컴포넌트 API 및 스타일 가이드 확인
- 컴포넌트가 노출하는 프로퍼티 값을 변경하며 동작확인
- 컴포넌트의 스냅샷을 이용해 UI 구조를 테스트
- 쉬운 GUI 인터페이스를 통한 엔지니어와 디자이너의 협업, 분업, 의사소통 촉진

**정적 타입 언어**

동적 타입 언어는 애플리케이션을 실행해 보기 전에는 애플리케이션이 기대대로 정상 동작할지 알 수가 없습니다.
개발팀이 여러 명으로 구성된 대규모 개발에서는 타입이 중요한 정보가 됨. 코드 타입이 애매하면 의도하지 않은 버그가 생길 수 있습니다.
견고성을 필요로 하는 수준 높은 웹 애플리케이션에 대한 수요가 높아지고 있습니다.
개발에 있어 일정 수준의 규모와 정확성이 요구되는 경우에는 정적 타입 언어가 더 적합합니다.

이런 필요에 부응해 Typescript를 시작으로 자바스크립트에도 정적 타입을 도입하려는 시도가 나타나고 있습니다.

## 7. Nuxt.js

Nuxt.js는 Vue.js의 서드파티 프로젝트입니다.

Nuxt.js를 이용하면 서버 사이드 렌더링(서버에서 UI를 렌더링하는 기능)을 지원하는 유니버설한 Vue.js 애플리케이션을 만들 수 있습니다.

복잡한 애플리케이션을 만드는 데 필요한 라우팅, 상태관리, 컴포넌트 관리 기능을 제공하는 모듈 등 완결된 스택을 갖춘 프레임워크입니다.
동시에 미들웨어나 플러그인으로 쉽게 확장할 수 있는 유연성도 가지고 있습니다.
애플리케이션 빌드, CLI 도구 등 개발 환경 구축 역시 Nuxt.js를 사용할 수 있습니다.

Nuxt.js는 대규모 웹 애플리케이션을 개발하는 데 있어 Vue.js를 선택하게 하는 이유중 하나입니다.

콘텐츠의 SEO/OGP 지원이 필요한 경우와 콘텐츠의 초기 로딩 시간을 단축해야하는 경우라면 서버 사이드 렌더링을 도입해야합니다.
(서버 운영 비용을 고려하면 서버 사이 렌더딩은 적용하지 않는 것이 현실적이지만 앞의 것이 꼭 필요한 애플리케이션이라면 도입해야합니다.)

## 8. 빌드 및 배포

`npm run buid`라는 명령어를 통해서 dist디렉토리 안에 정적파일로 어플리케이션이 출력되게 됩니다.

https://cli.vuejs.org/guide/deployment.html#platform-guides

github page, s3, netlify 등 여러 옵션으로 배포 가능하나 현업에서는 aws s3를 많이 사용하지 않을까 싶습니다.
