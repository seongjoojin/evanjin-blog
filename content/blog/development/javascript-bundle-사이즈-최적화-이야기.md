---
title: javascript bundle 사이즈 최적화 이야기
date: 2021-05-25 22:05:84
category: development
draft: false
---

## 왜 번들 사이즈를 줄이게 되었는가?

5월 부터 새로운 회사에 다니기 시작하였고 어드민 페이지를 작업하다보니 and design을 사용하게 되었고
다들 아시다 싶이 ant design은 기능이 많기 때문에 패키지 크기가 큰 편입니다.

그래서 번들 사이즈를 줄일 생각을 하게 되었습니다.

## 본격 번들 사이즈 줄이기 대작전

### Webpack Bundle Analyze를 통한 번들 크기 분석

`vue-cli-plugin-webpack-bundle-analyzer`라이브러리를 먼저 설치해줍니다.

```bash
vue add webpack-bundle-analyzer
```

설치 후 `vue.config.js`를 아래와 같이 설정 후 `localhost:8888`에서 확인하실 수 있습니다.

```js
module.exports = {
  pluginOptions: {
    webpackBundleAnalyzer: {
      openAnalyzer: false,
    },
  },
};
```

### ant-design-vue 라이브러리

먼저 ant design vue의 경우에는 공식 홈페이지에서 babel-plugin-import 플러그인을 설치하여서 최적화 될 수 있도록 안내해주고 있습니다.

https://2x.antdv.com/docs/vue/introduce/

하지만 회사의 서비스에서는 ant design vue의 component들의 스타일을 커스텀해서 사용하다보니 위의 설정대로 사용하면 만들어 놓은 커스텀 스타일이 먹지 않았기 때문에
Manually import를 하였습니다.

```ts
import DatePicker from "ant-design-vue/es/date-picker"
import "ant-design-vue/es/date-picker/style/css";
```

공식문서에서는 lib안에 있는 것을 사용하라고 하지만 es안에 있는 것이 사이즈가 더 작아서 es안에 있는 것을 사용하게 되었고 위와 같이 적용할 경우에도 커스텀 스타일이 제대로 작동되지 않는 면들이 있어서
필요한 ant design vue의 컴포넌트 css만 뽑아서 `main.ts`에 import하여서 사용하게 되었습니다.

### @ant-design/icons-vue 라이브러리

@ant-design/icons-vue는 아무래도 아이콘을 사용할 때 사용하게되었는데요. Webpack Bundle Analyze를 통해서 번들 사이즈를 분석해보니 사용하지 않는 것도 불러오게 되어 있었습니다.

어떤 방법이 있을지 고민하다가 `babel-plugin-transform-imports`플러그인을 사용하게 되었습니다.
해당 라이브러리로 import되는 것을 아래와 같이 ant-design-vue처럼 해줄 수 있지만 번거롭기도 하고 일일히 해주기에는 노가다로 보여서 사용하게 되었습니다. (ant-design-vue에선 되지 않아서 해당 라이브러리를 적용하진 않았습니다.)

```ts
import CheckCircleOutlined from '@ant-design/icons-vue/lib/icons/CheckCircleOutlined';
```

만약 메뉴얼 대로 하시고 싶으시면 위과 같이 하시면 됩니다.

`babel-plugin-transform-imports`를 설치 후에 `babel.config.js`의 아래 설정을 해주시면 위와 같이 import해주지 않아도 자동으로 위와 같이 import 하게 해줍니다.

```js
module.exports = {
  plugins: [
    [
      'transform-imports',
      {
        '@ant-design/icons-vue': {
          transform: '@ant-design/icons-vue/lib/icons/${member}',
          preventFullImport: true,
        },
      },
    ],
  ],
};
```

### moment 라이브러리

ant-design-vue에 moment 라이브러리가 사용되기 때문에 시간에 대한 라이브러리는 moment를 사용하게 되었습니다.

아무래도 moment는 용량이 큰 편으로 알려져 있었고 최적화 방법도 여러가지였지만 저는 locale를 제외하는 것으로 용량을 최적화 시켰습니다.

`babel.config.js`에서 아래와 같이 설정해주면 `moment`에서 locale이 제외되게 됩니다.

```js
const webpack = require('webpack');
module.exports = {
	configureWebpack: {
		plugins: [new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)],
	},
};
```

## 그래서 번들 사이즈 얼마나 줄였을까?

입사 후에 Webpack Bundle Analyze를 통해서 처음 번들 사이즈를 보았을 때 10mb 정도의 크기를 가지고 있었고 위의 세가지 방법을 통해서 3mb까지 줄이게 되었습니다.

## 참고할만한 것들

https://ui.toast.com/weekly-pick/ko_20190603
https://github.com/lodash/lodash-webpack-plugin
https://bundlephobia.com/
http://webpack.github.io/analyse/
