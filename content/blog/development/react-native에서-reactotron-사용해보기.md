---
title: react-native에서 reactotron 사용해보기
date: 2020-11-23 23:11:92
category: development
draft: false
---

## 왜 reactotron을 사용하는가

먼저 저는 RN디버깅을 하면서 Flipper를 주로 사용하였습니다만
어느 순간부터 ios시뮬레이터를 제대로 디버깅 못하는 현상이 발생하여서 다른 디버깅 툴을 사용하기로 하였습니다.

reactotron로 결정하게 된 이유는 제가 주로 사용하는 리덕스, 리덕스 사가, AsyncStorage의 동작을 다 볼 수 있기 때문이였습니다.

물론 Flipper와 다르게 프로젝트에 설치하고 설정해줘야하는 번거로움은 있지만 저는 Flipper가 ios 시뮬에서 안 되는 것이 불편하였기 때문에 그렇게 번거롭다고 생각하진 않았습니다.

## 설치

```bash
$ yarn add -D reactotron-react-native reactotron-redux reactotron-redux-saga
```

위의 세 라이브러리를 다 설치하신 후 `ReactotronConfig.js`를 생성해주신 후 아래와 같이 설정해줍니다.

```js
import AsyncStorage from '@react-native-async-storage/async-storage'
import Reactotron, { networking, openInEditor } from 'reactotron-react-native'
import { reactotronRedux } from 'reactotron-redux'
import sagaPlugin from 'reactotron-redux-saga'

const reactotron = Reactotron.configure()
  .useReactNative()
  .setAsyncStorageHandler(AsyncStorage)
  .use(reactotronRedux())
  .use(sagaPlugin())
  .use(networking())
  .use(openInEditor())
  .connect()

export default reactotron
```

다음으로 redux의 stroe 객체를 만드는 부분에서 아래와 같이 설정해주시면 됩니다.

```js
import { applyMiddleware, compose, createStore } from 'redux'
import createSagaMiddleware from 'redux-saga'
// ReactotronConfig.js의 위치는 다를 수 있습니다.
import Reactotron from '../../ReactotronConfig'

const sagaMonitor = Reactotron.createSagaMonitor()
const sagaMiddleware = createSagaMiddleware({ sagaMonitor })

const store = createStore(
  reducer,
  compose(applyMiddleware(sagaMiddleware), Reactotron.createEnhancer())
)
sagaMiddleware.run(rootSaga)

export default store
```
