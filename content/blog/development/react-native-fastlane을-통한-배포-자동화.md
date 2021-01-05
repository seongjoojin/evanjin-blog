---
title: react native fastlane을 통한 배포 자동화
date: 2020-10-17 16:10:09
category: development
draft: false
---

## 들어가는 말

React Native을 통해서 앱을 개발하던 중 테스트 및 프로덕트 배포시
api서버 설정을 잘 못하는 등의 사람의 실수로 인해서 문제가 될 때가 있었습니다.

현재는 저 같은 경우 jenkins, circle ci, travis ci 등으로 자동화되어 배포되지 않고 개발자의 맥북으로 배포되고 있습니다.

fastlane을 설정하면 jenkins(맥이 필요), circle ci, travis ci등을 통해서 배포자동화시 더욱 편하게 진행 가능합니다.

## fastlane 설치

먼저 xcode command line toools가 설치되어 있지 않다면 설치해줍니다.

```bash
$ xcode-select --install
```

다음으로 homebrew를 통하여서 설치해줍니다.

```bash
$ brew install fastlane
```

## fastlane 설정

다음으로 fastlane을 설정해줍니다.
ios, 안드로이드 각각 설정이 필요합니다.

### fastlane 설정(ios)

먼저 ios를 설정해주겠습니다.

```bash
$ cd ios
$ fastlane init
```

init시 어떤 용도로 fastlane을 사용할 것이지 물어보게 되는데요.

보통 ios는 테스트시 testflight를 사용하기 때문에
저는 2번인 Automate beta distribution to TestFlight를 선택해주었습니다.

선택 후 init이 완료되게 되는데요.
저희가 주로 다루게 될 파일은 `ios/fastlane/Fastfile`입니다.

FastFile을 여러분의 프로젝트의 설정대로 설정해주시면 되고 아래는 예시입니다.

```ruby
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :ios_beta do
    cert
    sigh(force: true)

    increment_build_number(xcodeproj: "example.xcodeproj")
    build_app(workspace: "example.xcworkspace", scheme: "example-staging")
    upload_to_testflight
  end
  desc "Push a new release build to the App Store"
  lane :ios_release do
    cert
    sigh(force: true)

    increment_build_number(xcodeproj: "example.xcodeproj")
    build_app(workspace: "example.xcworkspace", scheme: "example-prod")
    upload_to_testflight
  end
end
```

위에 대략적인 플로우는 아래와 같습니다.

먼저 `cert`를 통해서 인증서를 가져오고 `sigh(force: true)`로 프로버저닝 프로파일을 연결합니다.
그 후 build version을 올려줍니다.
마지막으로 빌드 후 testflight에 업로드해줍니다.

### fastlane 설정(안드로이드)

fastlane을 설정하기 전에 google play console에서 service account를 생성해주어야 합니다.
이 부분은 제가 스크린샷을 남기지 못하여서 아래 블로그에서 참고해주시면 됩니다.

https://dev-yakuza.github.io/ko/react-native/fastlane/#api-access%EB%A5%BC-%EC%9C%84%ED%95%9C-service-account-%EC%83%9D%EC%84%B1

그 후 fastlane 초기 설정을 해줍니다.

```bash
$ cd android
$ fastlane init
```

다음으로 `android/fastlane/Fastfile`을 프로젝트 설정대로 수정해줍니다.

예시

```ruby
default_platform(:android)

platform :android do

  desc "Submit a new Google Beta Build Apk"
  lane :google_beta do
    gradle(
      task: 'assemble',
      flavor: "google",
      build_type: 'Staging'
    )
  end
  desc "Make new Google Release App Bundle"
  lane :google_release do
    gradle(
      task: 'bundle',
      flavor: "google",
      build_type: 'Release'
    )
  end
end
```

위와 같이 apk, 번들 파일을 수정 후 google play나 Firebase App Distribution, deploygate 등에 업로드 가능합니다.

베타 테스트의 경우 아래의 설정을 참고해주시면 됩니다.

https://docs.fastlane.tools/getting-started/android/beta-deployment/#building-your-app
https://docs.fastlane.tools/actions/deploygate/#deploygate

릴리즈의 경우 아래의 설정을 참고해주시면 됩니다.
예시는 apk파일인데 app bundle로는 테스트해보지 못해서 되는지는 검증해보지 못하였습니다.

https://docs.fastlane.tools/getting-started/android/release-deployment/#uploading-your-apk

## fastlane을 통한 배포

위와 같이 Fastfile을 수정 후 아래의 명령어들로 배포 자동화 및 apk, bundle 등을 생성해줄 수 있습니다.

```bash
# ios beta
$ cd ios
$ fastlane ios_beta
# ios release
$ cd ios
$ fastlane ios_release
# aos beta
$ cd android
$ fastlane google_beta
# aos release
$ cd android
$ fastlane google_release
```

ios 배포시 https://appleid.apple.com/account/manage 에서 암호생성해서 넣어달라는 메세지가 나올때가 있습니다.
이때에 위의 링크로 들어가서 ios 개발자 계정으로 로그인해준 후 보안 앱 암호에서 암호를 생성하여서 넣어주시면 됩니다.

## 유용한 플러그인들 소개

fastlane의 또 다른 장점은 유용한 플러그인들이 유저에 의해서 생성되었다는 점입니다.
(https://docs.fastlane.tools/plugins/available-plugins/)

저에게 유용했던 몇가지 플러그인들을 소개드립니다.

먼저 yarn plugin입니다.
https://github.com/joshrlesch/fastlane-plugin-yarn

제가 만든 앱의 경우 `package.json`에 넣은 `scripts`를 통해서 배포환경 변수를 컨트롤 하는 부분이 있기 때문에
그것을 배포하기전에 한 번 설정해줄 수 있도록 해당 플러그인을 사용하였습니다.

다음으로 `increment_version_code plugin`입니다.
https://github.com/Jems22/fastlane-plugin-increment_version_code

fastlane에서 ios는 빌드버전을 올려주는 것이 들어가 있으나 안드로이드에서는 없지만 위의 플러그인으로 해줄 수 있습니다.

그외에도 많은 플러그인들이 존재하니 여러분들의 프로젝트에 특성에 따라서 입맛대로 설정해주시면 됩니다.

## 맺는말

먼저 fastlane의 존재를 알게해주신 dev yakuza께 감사를 먼저드립니다.

fastlane 이전에는 개발자들이 yarn으로 환경변수 세팅 후 xcode 이나 안드로이드 스튜디오를 실행 후 스키마나 그래들 환경 변수 설정 후 빌드를 일일히 해주었는데
fastlane을 설정 후 명령어 하나로 자동으로 TestFlight, deploygate에 업로드 되거나 앱번들이 생성되는 것 등이 자동화 됨으로써 개발자의 실수를 줄이고 시간을 단축하게 되었습니다. (안드로이드 배포의 경우 3-4분정도로 엄청나게 단축하게 되었습니다.)

감사합니다.
