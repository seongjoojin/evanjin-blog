---
title: React Native Webview 사용기
date: 2020-07-14 23:07:28
category: development
draft: false
---

## 들어가는 말

RN에서 Webview로는 postMessage로 값을 보낼 수 있으며
Webview에서 RN로는 `window.ReactNativeWebView.postMessage`로 값을 보낼 수 있습니다.

위와 같은 식으로 RN과 Webview간에 양방향 통신이 가능합니다.

## React Native에서의 구성

```js
import React, { useRef } from 'react'
import WebView from 'react-native-webview'

const onMessage = (e) => {
  const event = JSON.parse(e.nativeEvent.data)
  console.log('onMessage', event)
}

const WebViewContainer = () => {
  return <WebView ref={webViewRef} onMessage={onMessage} />
}

export default WebViewContainer
```

먼저 React Native에서는 위와 같이 onMessage를 설정하고 WebView -> RN으로 보내주는 값을 받을 준비를 합니다.

## 웹뷰(React)에서의 구성

웹뷰를 이룰 수 있는 것들은 많은 것으로 만들 수 있지만 저는 React로 구성하였으니 React로 예시를 들겠습니다.

```js
import React, { useEffect } from 'react'

const Component = () => {
  const onMessageHandler = (e) => {
    const event = JSON.parse(e.data)
    window.ReactNativeWebView.postMessage(JSON.stringify({ event: event }))
  }

  useEffect(() => {
    const isUIWebView = () => {
      return navigator.userAgent
        .toLowerCase()
        .match(/\(ip.*applewebkit(?!.*(version|crios))/)
    }

    const receiver = isUIWebView() ? window : document

    receiver.addEventListener('message', onMessageHandler)
    return () => {
      receiver.removeEventListener('message', onMessageHandler)
    }
  })

  return (
    <div>
      <p>Hello</p>
      <button>Button</button>
    </div>
  )
}

export default Component
```

onMessageHandler를 위와 같이 설정하여서 RN -> Webview로 주는 값을 받을 준비를 합니다.

## 예시

먼저 간단하게 RN->Webview로 값을 넘겨서 글자의 텍스트를 변화를 주는 예시를 들어보겠습니다.

먼저 RN 코드입니다.

```js
import React, { useRef } from 'react'
import WebView from 'react-native-webview'

const onMessage = (e) => {
  const event = JSON.parse(e.nativeEvent.data)
  console.log('onMessage', event)
}

const onLoadProgress = ({ nativeEvent }) => {
  if (nativeEvent.progress === 1) {
    if (webViewRef.current) {
      webViewRef.current.postMessage(
        JSON.stringify({ changeText: 'World' }),
        '*'
      )
    }
  }
}

const WebViewContainer = () => {
  return (
    <WebView
      ref={webViewRef}
      onMessage={onMessage}
      onLoadProgress={onLoadProgress}
    />
  )
}

export default WebViewContainer
```

웹뷰가 다 로딩된 후에 postMessage로 값을 보내주도록 하였습니다.
postMessage에는 string만 보낼 수 있기 때문에 JSON.stringify로 변환하여서 보내줍니다.

```js
import React, { useEffect, useState } from 'react'

const [textValue, setTextValue] = 'Hello'

const Component = () => {
  const onMessageHandler = (e) => {
    const event = JSON.parse(e.data)
    window.ReactNativeWebView.postMessage(JSON.stringify({ event: event }))
    if (event.changeText) {
      setTextValue(event.changeText)
    }
  }

  useEffect(() => {
    const isUIWebView = () => {
      return navigator.userAgent
        .toLowerCase()
        .match(/\(ip.*applewebkit(?!.*(version|crios))/)
    }

    const receiver = isUIWebView() ? window : document

    receiver.addEventListener('message', onMessageHandler)
    return () => {
      receiver.removeEventListener('message', onMessageHandler)
    }
  })

  return (
    <div>
      <p>{textValue}</p>
      <button>Button</button>
    </div>
  )
}

export default Component
```

WebView(React) 코드입니다.
RN에서 보낸 메세지를 만들어둔 onMessageHandler로 통해서 받은 후 위와 같이 처리합니다.

다음으로는 Webview에서 RN으로 값을 보내는 예시를 진행하겠습니다.

```js
import React, { useEffect, useState } from 'react'

const [textValue, setTextValue] = 'Hello'

const Component = () => {
  const onMessageHandler = (e) => {
    const event = JSON.parse(e.data)
    window.ReactNativeWebView.postMessage(JSON.stringify({ event: event }))
    if (event.changeText) {
      setTextValue(event.changeText)
    }
  }

  const onClick = (e) => {
    e.preventdefault()
    window.ReactNativeWebView.postMessage(JSON.stringify({ method: 'click' }))
  }

  useEffect(() => {
    const isUIWebView = () => {
      return navigator.userAgent
        .toLowerCase()
        .match(/\(ip.*applewebkit(?!.*(version|crios))/)
    }

    const receiver = isUIWebView() ? window : document

    receiver.addEventListener('message', onMessageHandler)
    return () => {
      receiver.removeEventListener('message', onMessageHandler)
    }
  })

  return (
    <div>
      <p>{textValue}</p>
      <button onClick={onClick}>Button</button>
    </div>
  )
}

export default Component
```

먼저 WebView(React) 코드입니다.
버튼 누를시 WebView -> RN으로 값이 가도록 하였습니다.

window.ReactNativeWebView.postMessage도 string 형태로 값을 보내기에 JSON.stringify로 변환해주어야 합니다.

```js
import React, { useRef } from 'react'
import WebView from 'react-native-webview'

const onMessage = (e) => {
  const event = JSON.parse(e.nativeEvent.data)
  console.log('onMessage', event)
  if (event.method === 'click') {
    //이벤트 발생
  }
}

const onLoadProgress = ({ nativeEvent }) => {
  if (nativeEvent.progress === 1) {
    if (webViewRef.current) {
      webViewRef.current.postMessage(
        JSON.stringify({ changeText: 'World' }),
        '*'
      )
    }
  }
}

const WebViewContainer = () => {
  return (
    <WebView
      ref={webViewRef}
      onMessage={onMessage}
      onLoadProgress={onLoadProgress}
    />
  )
}

export default WebViewContainer
```

RN 코드입니다.
onMessage에 WebView로부터 보낸 PostMessage를 받을 수 있으며 위의 코드와 같이 받을 수 있습니다.
