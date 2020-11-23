---
title: react-native에서 patch-package 사용해보기
date: 2020-11-23 22:11:08
category: development
draft: false
---

## patch-package란?

우리는 프로그램을 만들면서 여러가지 오픈소스들을 사용합니다.
오픈소스를 쓰다보면 메인테이너가 유지보수를 포기해서 고칠 필요가 있거나 당장의 오류가 나는 부분을 고쳐야 할 필요가 생길때가 있습니다.

깃허브에 포크 후 수정하여도 되지만 수정 후에 바로 실행해보기 번거로운 면이 있습니다.

저는 주로 patch-package를 사용합니다.

## 사용법

공식 깃허브에도 잘 나와있지만 대략적으로 사용법을 정리해보도록 하겠습니다.

먼저 설치부터 하도록 하겠습니다.

```bash
$ yarn add patch-package postinstall-postinstall
```

설치 후에 package.json안의 scripts에 아래 명령어를 추가해줍니다.

```json
"scripts": {
  "postinstall": "patch-package"
}
```

다음으로는 node_modules 아래에 고쳐야할 패키지를 수정 후 아래와 같이 명령어를 실행해줍니다.

```bash
$ yarn patch-package package-name
```

위와 같이 실행하면 patchs 폴더 아래에 patch파일들이 생성되게 됩니다.

제가 patch-package로 수정한 react-native-kakao-link를 보시면 아래와 같이 생성되었습니다.

```patch
diff --git a/node_modules/react-native-kakao-links/index.d.ts b/node_modules/react-native-kakao-links/index.d.ts
index 2a99ecf..8c75586 100644
--- a/node_modules/react-native-kakao-links/index.d.ts
+++ b/node_modules/react-native-kakao-links/index.d.ts
@@ -1,3 +1,6 @@
-declare function link(options: Object): Promise<Object>;
-
-export default link;
+declare module 'react-native-kakao-links' {
+  const RNKakaoLink: {
+    link(options: Object): Promise<Object>;
+  }
+  export default RNKakaoLink;
+}
\ No newline at end of file
diff --git a/node_modules/react-native-kakao-links/ios/RNKakaoLink.podspec b/node_modules/react-native-kakao-links/ios/RNKakaoLink.podspec
index f921839..3b45cdb 100644
--- a/node_modules/react-native-kakao-links/ios/RNKakaoLink.podspec
+++ b/node_modules/react-native-kakao-links/ios/RNKakaoLink.podspec
@@ -17,7 +17,7 @@ Pod::Spec.new do |s|


   s.dependency "React"
-  # s.dependency 'KakaoOpenSDK'
+  s.dependency 'KakaoOpenSDK'
   #s.dependency "others"

 end
```

저는 타입이랑 sdk 설정 수정이 필요해서 위와 같이 설정하였고
결국 - 부호로 된 것들은 패키지를 설치시에 +부호인 걸로 변경되는 것임을 보실 수 있습니다.
