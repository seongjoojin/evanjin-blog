---
title: react native sentry 이용팁
date: 2020-10-17 15:10:78
category: development
draft: false
---

## 들어가는 말

앱을 개발 후 배포 후 유저로 부터 버그 제보를 받는 경우가 많아지게 되면
이런 것들에 대해서 어떻게 대응해야할지 고민을 하게 됩니다.

저도 React Native로 개발한 앱에서 어떻게 하면 이런 버그들에 대해서 대응하기 쉬워질까 하다가 찾은 것이 sentry였습니다.

만약 다른 솔루션을 원하신다면 bugsnag(https://www.bugsnag.com/)을 이용하시면 됩니다.

## sourcemap 업로드

설치부터 정리해보려고 할까 하였지만 너무나도 쉽게 잘 되기 때문에 특별히 언급해드리지 않아도 잘하실거 같아서 생략하였습니다.
(설치는 https://docs.sentry.io/platforms/react-native/ 이 링크에서 참고하시면 됩니다.)

소스맵을 자동으로 올려주지만 누락되는 경우가 종종 발생하기 때문에 수동으로 올려야할 때가 많습니다.

그래서 이용팁 중에 먼저 소스맵을 수동으로 업로드 방법을 먼저 알려드리려고 합니다.

### sentry cli 설치 및 계정 로그인

먼저 sentry cli를 설치해줍니다.

```bash
$ brew install getsentry/tools/sentry-cli
```

설치 후 sentry 로그인을 진행해줍니다.

```bash
$ sentry-cli login
```

### 번들 생성

안드로이드, ios 각각의 번들을 생성해줍니다.

먼저 ios 번들을 생성해줍니다.

```bash
$ npx react-native bundle \
  --dev false \
  --platform ios \
  --entry-file index.js \
  --bundle-output main.jsbundle \
  --sourcemap-output ios.main.bundle.map
```

먼저 안드로이드 번들을 생성해줍니다.

```bash
$ npx react-native bundle \
  --dev false \
  --platform android \
  --entry-file index.js \
  --bundle-output android.main.bundle \
  --sourcemap-output android.main.bundle.map
```

### 번들 업로드

마지막으로 생성한 번들을 sentry cli를 통하여서 업로드해줍니다.

```bash
$ sentry-cli releases -o organizationName -p projectName files version upload-sourcemaps --rewrite android.main.bundle.map ios.main.bundle.map
```

예시

```bash
$ sentry-cli releases -o example -p example files 1.0.0 upload-sourcemaps --rewrite android.main.bundle.map ios.main.bundle.map
```

## 터치 이벤트 트래킹

sentry를 사용하면 어떤 api를 불렀는지 어떤 오류가 나왔는지는 보기 좋으나
한가지 아쉬울 때가 어떤 컴포넌트 (button 등)을 사용자가 터치하였을때 그랬는지 알고 싶을 때가 있습니다.

설정하기 전에 알아야할 것은 메트로 설정으로 클래스와 함수이름이 다 나오도록 설정해야하므로 번들사이즈가 늘어가게 된다는 것입니다.

### 터치 이벤트 트래킹 설정

```js
import * as Sentry from "@sentry/react-native";

const App = () => {
  return (
    <Sentry.TouchEventBoundary>
      <Example />
    </Sentry.TouchEventBoundary>
  );
};

export default App;
```

저 같은 경우는 해당 이벤트를 App.tsx(App.js)에 설정해주었습니다.
index.js에서 설정해주어도 되며 전체앱을 감싸주기만 하면 됩니다.

hoc로도 설정가능하나 제가 hoc를 좋아하지 않아서 위와 같이 설정해주었습니다.

### 메트로 설정

위에 말씀드렸던 클래스와 함수이름이 나오도록 설정하여야 해당 이벤트를 설정한 의미가 있게 됩니다.

`metro.config.js`에 아래와 같은 설정을 추가해줍니다.

```js
module.exports = {
  transformer: {
    minifierConfig: {
      keep_classnames: true,
      keep_fnames: true,
      mangle: {
        keep_classnames: true,
        keep_fnames: true,
      },
    },
  },
};
```

터치 이벤트 트래킹에 대한 더 자세한 것이 필요하시다면 공식 문서를 참고해주시면 됩니다.
(https://docs.sentry.io/platforms/react-native/touchevents/)

## 네비게이션 이동 표시

sentry 사용시 사용자가 호출하는 api로도 충분히 어떤 화면에서 에러가 났는지 인지할 수 있지만
가끔은 어려운 경우가 많습니다.

그래서 사용자가 어떻게 화면을 이동하는지가 궁금한 경우가 생깁니다.

공식문서에서 해당 방법을 찾던 중 Breadcrumbs를 사용하면 되겠다는 생각이 들었습니다.
(https://docs.sentry.io/product/error-monitoring/breadcrumbs/)

그럼 어떻게 제가 사용하였는지 이제 알아보겠습니다.

### Breadcrumbs 적용

저는 react-navigation을 사용중이고 react-navigation에서 스택, 탭 등의 네비게이션을 생성시 NavigationContainer 안에 생성되게 되는데요.
NavigationContainer의 메소드 중 onStateChange를 사용하여서 구현하였습니다.

```js
import React, {useState} from 'react';
import * as Sentry from "@sentry/react-native";
import {NavigationContainer} from '@react-navigation/native';

const App = () => {
  const [currentRouteName, setCurrentRouteName] = useState('initRouteName');
  return (
    <NavigationContainer
      onStateChange={(state) => {
        Sentry.addBreadcrumb({
          category: 'Navigation',
          data: {
            to: `${state?.routes[state.index].name}`,
            from: `${currentRouteName}`,
          },
          level: Sentry.Severity.Info,
        });
        setCurrentRouteName(state?.routes[state.index].name);
      }}
    >
      ...
    </NavigationContainer>
  );
};

export default App;
```

## 태그 설정

마지막으로 태그 설정하는 방법을 알아보고 마무리하려고 합니다.

태그는 discover에서 검색할 수 있으며 issues의 상세 페이지에 표기 됨으로 어떤 에러 구분등을 구분하여서 유용하게 사용할 수 있습니다.

예시를 들면 아래와 같이 설정이 가능합니다.

```js
Sentry.withScope((scope) => {
  scope.setTag('error type', 'API Error');
  scope.setTag('error api uri', `${url}`);
  Sentry.captureException(error);
});
```

### 맺는 말

React Native에서 Sentry를 사용하면서 팁으로 정리해놓을 것을 블로그로 옮겨봤습니다.
조금이라도 도움이 되시길 바랍니다.

감사합니다.