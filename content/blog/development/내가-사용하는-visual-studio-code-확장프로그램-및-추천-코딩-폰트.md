---
title: 내가 사용하는 Visual Studio Code 확장프로그램 및 추천 코딩 폰트
date: 2019-09-22 14:09:29
category: development
---

## Visual Studio Code 추천 확장프로그램

### [Vue Development Extension Pack](https://marketplace.visualstudio.com/items?itemName=changjoo-park.vscode-vue-devpack)

처음 프론트 입문을 vue로 시작해서 IDE를 atom에서 vscode로 넘어오고 vue에 관한 확장프로그램을 찾다가 위의 것을 찾게되었고 지금도 잘 사용하고 있습니다.

일단 위에 확장프로그램에서 vue에 관련된 것만 살펴보면 아래와 같습니다.

#### [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur)

vue를 사용한다면 이 확장프로그램은 거의 필수이지 않을까 싶습니다.
highlighting, Snippet 등 vue의 sfc파일을 생성하고 코드를 생성하는데 매우 도움 되는 기능을 가지고 있습니다.

#### [Vue Peek](https://marketplace.visualstudio.com/items?itemName=dariofuzinato.vue-peek)

vue는 컴포넌트를 sfc(.vue)로 만들게됩니다.
이를 추적할 수 있도록 해주는 확장프로그램입니다.
일일히 폴더를 찾지 않고 클릭 한 번이면 해당 vue파일로 이동하게 되니 유용합니다.

다음으로 git에 관해 유용한 확장 프로그램은 아래와 같습니다.

#### [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)

파일에서 바로 git에 언제 커밋했고 누가 하였는지 내역을 바로 볼 수 있습니다.

#### [Git History](https://marketplace.visualstudio.com/items?itemName=donjayamanne.githistory)

하나의 파일 혹은 프로젝트의 git의 내역을 볼 수 있는 확장 프로그램입니다.

#### [Git Project Manager](https://marketplace.visualstudio.com/items?itemName=felipecaputo.git-project-manager)

로컬에 저장되어 있는 git 프로젝트 폴더에 바로 접근하기 쉽도록 도와주는 확장프로그램입니다.

마지막으로 나머지 제가 유용하게 사용하는 편인 확장프로그램을 소개하도록 하겠습니다.

#### [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)

협업을 할 때 각자 다른 vscode의 환경으로 진행하다보면 eslint 등을 맞추기 어려울 때가 있는데 이런 부분을 해결해주는 확장프로그램 입니다.

#### [npm](https://marketplace.visualstudio.com/items?itemName=eg2.vscode-npm-script)

npm 스크립트 실행을 도와주는 확장 프로그램입니다.

#### [npm Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.npm-intellisense)

npm module들을 import, require하기 쉽게 도와주는 확장프로그램입니다.

#### [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)

컴포넌트나 함수 등을 따로 파일로 분리하고 다른 파일에 import하고 싶을 때 해당 파일의 위치를 찾기 어려울 때가 많습니다.이런 어려움을 도와주는 확장프로그램입니다.
거의 필수 확장프로그램입니다.

#### [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)

http 확장자로 작성하여 해당 rest api가 제대로 동작하는지와 받았을 때의 변수들은 무엇인지 쉽게 체크할 수 있는 확장프로그램입니다.

#### [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)

프로그램을 하다보면 중괄호가 어디서 닫히고 어디서 시작되는지 어려울 때가 있습니다.
특히 요새 뜨고 있는 dart를 하다보면 양파껍질 같은 중괄호를 볼 수 있는데요.
중괄호들을 색상으로 구분하여 프로그래머가 조금 더 쉽게 프로그래밍 할 수 있도록 도와줍니다.

#### [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)

해당 확장프로그램은 설치한 라이브러리들을 import나 require할 때 해당하는 라이브러리들의 용량을 표시해줍니다.

이제 제가 개인적으로 잘 쓰고 있는 확장프로그램을 살펴보도록 하겠습니다.

### [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)

js, json, css, sass, jsx, vue 등의 파일들의 포맷을 아름답게 맞춰주는 확장프로그램입니다.
전에는 페이지가서 변환하는 등의 과정을 거쳤지만 해당 프로그램을 설치 후 아래와 같이 설정한다면 파일을 저장할 때마다 맞춰주게 됩니다.

https://velog.io/@velopert/eslint-and-prettier-in-react

### [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

프로그래머들은 각자마다 다른 코딩 스타일을 가지고 있습니다.
예를 들면 tab을 쓰냐 스페이스바를 쓰냐 작은 따옴표냐 큰 따옴표냐 등의 이야기입니다.
보기에는 아주 사소해보입니다만 이러한 것은 미묘한 분란을 일으키기 좋은 것들이라고 생각합니다.
이러한 분란들을 없애기 위해서 사내의 코드 컨벤션을 만들고 이 컨벤션에 맞지 않으면 안되도록 강제해주는 툴이라고 보시면 됩니다.

### [DotENV](https://marketplace.visualstudio.com/items?itemName=mikestead.dotenv)

프로그래밍을 하다보면 숨겨하는 변수들이 있습니다. aws키나 DB에 접근하기 위한 정보 등은 보통 노출되면 치명적인 문제를 일으킬 수 있는 것들 입니다.

이러한 것들을 숨기기 위해 DotENV라는 라이브러리를 사용하게 됩니다.
이 라이브러리에서 보통 .dotenv의 파일을 사용하게 되는 이 파일의 문법 강조를 해주는 역할을 합니다.

### [indent-rainbow](https://marketplace.visualstudio.com/items?itemName=oderwat.indent-rainbow)

중괄호와 함께 indent는 프로그래머가 인식하기 어려운 존재중 하나입니다.
이 indent가 제대로 되었는지와 식별하기 쉽게 색깔로 구분해 주는 확장프로그램입니다.

### [vscode-styled-components](https://marketplace.visualstudio.com/items?itemName=jpoissonnier.vscode-styled-components)

제가 리액트를 사용하면 가장 좋아하는 것이 styled-components였는데 딱하나 불편한 것이 자동완성이나 문법 강조가 잘 되지 않는 면이 있었습니다.
이러한 면들을 속 시원하게 긁어주는 확장프로그램입니다.

### [Settings Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync)

제가 가장 좋아하는 확장프로그램 중 하나입니다.
webstorm 같은 IDE는 같은 계정이면 사용하는 설정을 어디서든 동기화할 수 있습니다.
이러한 점은 새로운 회사를 가거나 포맷을 하였을시 자기가 사용하던 환경을 그대로 사용할 수 있다는 장점을 가지고 있습니다.
vscode에서 그렇게 할 수 있는 확장프로그램을 찾다가 Settings Sync를 찾게되었습니다.
해당 확장프로그램은 사용자의 gist를 사용해서 gist에 올린 설정을 어디서든 다운 받아서 동기화할 수 있도록 해줍니다.

마지막으로 제가 유용하게 사용하는 코딩 폰트들을 소개합니다.

## 추천 코딩 폰트

### [D2 Coding](https://github.com/naver/d2codingfont)

네이버 D2에서 내놓은 코딩을 위한 폰트입니다.
개인적으로는 거의 2년 정도 사용했었고 영문자, 숫자, 한글, 특수 기호들이 거의 명확하게 다르게 표현하는 것과 0의 대해서 표현하는 것도 너무 마음에 들어서 계속 사용 중에 있습니다.

### [FiraCode](https://github.com/tonsky/FiraCode)

FiraCode 같은 경우 지인이 쓰신다고 해서 한번 사용해본 폰트입니다.
!=, !==, === 등의 표현이 특이하고 화살표도 이 폰트만의 특이한 표현으로 표현해내는 점이 독특합니다.
가독성이 괜찮은 폰트라서 코딩폰트로써 사용해도 괜찮은 폰트라고 생각합니다.
