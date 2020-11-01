---
title: husky와 lint-staged 사용해보기
date: 2020-11-01 13:11:64
category: development
draft: false
---

## 들어가는 말

eslint와 prettier를 사용하면서 같이 협업을 하다보면
eslint와 prettier 법칙에 맞지 않는 코드가 올라올 경우도 발생하게 됩니다.

그럴 때 필요한 것이 huksy와 line-staged입니다.

husky는 git에서 특정 이벤트(커밋, 푸쉬 등)을 실행할 때 이벤트에 hooks를 설정하여 hooks에 설정된 스크립트를 실행할 수 있습니다.

line-staged는 git의 staged된 상태에 파일들에 특정 명령어를 실행할 수 있도록 해주는 도구이며
line-staged는 Staged된 파일을 수정한 후 다시 git add를 하지 않아도 문제가 없도록 도와주는 도구 입니다.

## 설치

```bash
$ yarn add -D husky lint-staged
```

```bash
$ npm i -D husky lint-staged
```

## 설정

기본적으로는 둘 다 package.json안에서 정의할 수 있습니다.

```json
"lint-staged": {
  "src/**/*.{ts,tsx}": [
    "eslint --ext .tsx,ts src/ --fix"
  ],
  "src/**": [
    "prettier --check 'src/**/*.ts' 'src/**/*.tsx'"
  ]
},
"husky": {
  "hooks": {
    "pre-commit": "lint-staged"
  }
}
```

위의 예시에서는 husky를 통해서 깃 커밋되기 전에 line-staged이 실행될 수 있도록 하였고

line-staged 설정에서 설정한 파일들이(`src/**/*.{ts,tsx}/src/**`) 변했을 경우
설정한 eslint와 prettier 명령어가 실행되게 됩니다.

만약 eslint와 prettier cli 명령어에 대해서 아시고 싶다면 공식문서를 참고해주시면 될거 같습니다.

https://eslint.org/docs/user-guide/command-line-interface
https://prettier.io/docs/en/cli.html

lint-staged 같은 경우 `.lintstagedrc` 파일로 따로 설정해주실 수 있습니다.
