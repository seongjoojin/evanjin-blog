---
title: react-native에서 expo 라이브러리 사용해보기
date: 2020-11-24 00:11:89
category: development
draft: false
---

## 왜 expo 라이브러리를 사용하는가

당연히 서비스에 필요했기 때문이기도 하고
아무래도 오픈소스들의 불안한 유지보수의 면모들도 expo가 어느정도 나은 면이 있다고 판단되었습니다.

일단 제가 만드는 서비스에는 화면 캡쳐, 녹화를 막는 라이브러리가 필요했으나 아무라 찾아봐도 없어서 네이티브 모듈로 만들려고 했으나
아직 네이티브 지식이 부족해서 더 찾다보니 expo내에서 화면캡쳐, 녹화 방지 라이브러리가 있어서 사용하게 되었습니다.
(https://docs.expo.io/versions/v39.0.0/sdk/screen-capture/)

## react-native-unimodule 설치

먼저 expo 라이브러리를 설치하기 위해서는 react-native-unimodules 라이브러리를 설치해야합니다.
(https://docs.expo.io/bare/installing-unimodules/)

위의 페이지에 잘 나와있지만 굳이 정리해보자면 아래와 같습니다.
먼저 `react-native-unimodules`를 설치합니다.

```bash
$ yarn add react-native-unimodules
$ npx pod-install
```

다음으로 네이티브 설정을 해줍니다.

`ios/MyApp/AppDelegate.h`

```diff-objectivec
  #import <React/RCTBridgeDelegate.h>
  #import <UIKit/UIKit.h>

- @interface AppDelegate : UIResponder <UIApplicationDelegate, RCTBridgeDelegate>
+ #import <UMCore/UMAppDelegateWrapper.h>

+ @interface AppDelegate : UMAppDelegateWrapper <UIApplicationDelegate, RCTBridgeDelegate>

  @property (nonatomic, strong) UIWindow *window;
```

`ios/MyApp/AppDelegate.m`

```diff-objectivec
  #import <React/RCTBridge.h>
  #import <React/RCTBundleURLProvider.h>
  #import <React/RCTRootView.h>

+ #import <UMCore/UMModuleRegistry.h>
+ #import <UMReactNativeAdapter/UMNativeModulesProxy.h>
+ #import <UMReactNativeAdapter/UMModuleRegistryAdapter.h>

  #ifdef FB_SONARKIT_ENABLED
  #import <FlipperKit/FlipperClient.h>
  #import <FlipperKitLayoutPlugin/FlipperKitLayoutPlugin.h>
  #import <FlipperKitUserDefaultsPlugin/FKUserDefaultsPlugin.h>
  #import <FlipperKitNetworkPlugin/FlipperKitNetworkPlugin.h>
    [client addPlugin:[[FlipperKitNetworkPlugin alloc] initWithNetworkAdapter:[SKIOSNetworkAdapter new]]];
    [client start];
  }
  #endif

+ @interface AppDelegate () <RCTBridgeDelegate>
+
+ @property (nonatomic, strong) UMModuleRegistryAdapter *moduleRegistryAdapter;
+
+ @end

  @implementation AppDelegate

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
  {
  #ifdef FB_SONARKIT_ENABLED
    InitializeFlipper(application);
  #endif

+   self.moduleRegistryAdapter = [[UMModuleRegistryAdapter alloc] initWithModuleRegistryProvider:[[UMModuleRegistryProvider alloc] init]];

    RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:self launchOptions:launchOptions];
    RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                    moduleName:@"MyApp"
                                              initialProperties:nil];

    self.window.rootViewController = rootViewController;
    [self.window makeKeyAndVisible];
+   [super application:application didFinishLaunchingWithOptions:launchOptions];
    return YES;
  }

+ - (NSArray<id<RCTBridgeModule>> *)extraModulesForBridge:(RCTBridge *)bridge
+ {
+     NSArray<id<RCTBridgeModule>> *extraModules = [_moduleRegistryAdapter extraModulesForBridge:bridge];
+     // If you'd like to export some custom RCTBridgeModules that are not Expo modules, add them here!
+     return extraModules;
+ }
+
  - (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
+
  {
  #if DEBUG
    return [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
  #else
    return [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
```

`ios/Podfile`

```diff-ruby
  require_relative '../node_modules/react-native/scripts/react_native_pods'
  require_relative '../node_modules/@react-native-community/cli-platform-ios/native_modules'
+ require_relative '../node_modules/react-native-unimodules/cocoapods.rb'

  platform :ios, '10.0'

  target 'MyApp' do
+   use_unimodules!
    config = use_native_modules!

    use_react_native!(:path => config["reactNativePath"])

  target 'MyAppTests' do
```

다음으로 android를 설정해줍니다.

`android/app/build.gradle`

```diff-groovy
  apply plugin: "com.android.application"
+ apply from: '../../node_modules/react-native-unimodules/gradle.groovy'

  import com.android.build.OutputFile

  implementation "com.facebook.react:react-native:+"  // From node_modules

  implementation "androidx.swiperefreshlayout:swiperefreshlayout:1.0.0"

+ addUnimodulesDependencies()
+
  debugImplementation("com.facebook.flipper:flipper:${FLIPPER_VERSION}") {
    exclude group:'com.facebook.fbjni'
  }

  debugImplementation("com.facebook.flipper:flipper-network-plugin:${FLIPPER_VERSION}") {
```

`android/app/src/main/java/com/myapp/MainApplication.java`

```diff-java
  package com.myapp;

+ // com.myapp should be your package name
+ import com.myapp.generated.BasePackageList;
+
  import android.app.Application;
  import android.content.Context;
  import com.facebook.react.PackageList;
  import com.facebook.react.ReactApplication;
  import com.facebook.react.ReactInstanceManager;
  import com.facebook.react.ReactNativeHost;
  import com.facebook.react.ReactPackage;
  import com.facebook.soloader.SoLoader;
  import java.lang.reflect.InvocationTargetException;
  import java.util.List;
+ import java.util.Arrays;
+
+ import org.unimodules.adapters.react.ModuleRegistryAdapter;
+ import org.unimodules.adapters.react.ReactModuleRegistryProvider;
+ import org.unimodules.core.interfaces.SingletonModule;

  public class MainApplication extends Application implements ReactApplication {
+   private final ReactModuleRegistryProvider mModuleRegistryProvider = new ReactModuleRegistryProvider(new BasePackageList().getPackageList(), null);

    private final ReactNativeHost mReactNativeHost =
        new ReactNativeHost(this) {
          @Override
          public boolean getUseDeveloperSupport() {
          protected List<ReactPackage> getPackages() {
            @SuppressWarnings("UnnecessaryLocalVariable")
            List<ReactPackage> packages = new PackageList(this).getPackages();
            // Packages that cannot be autolinked yet can be added manually here, for example:
            // packages.add(new MyReactNativePackage());
+
+           // Add unimodules
+           List<ReactPackage> unimodules = Arrays.<ReactPackage>asList(
+             new ModuleRegistryAdapter(mModuleRegistryProvider)
+           );
+           packages.addAll(unimodules);
            return packages;
          }

          @Override
          protected String getJSMainModuleName() {
```

`android/build.gradle`

```diff-groovy
  buildscript {
      ext {
          buildToolsVersion = "29.0.2"
-         minSdkVersion = 16
+         minSdkVersion = 21
          compileSdkVersion = 29
          targetSdkVersion = 29
      }
      repositories {
          google()
```

`android/settings.gradle`

```diff-groovy
  rootProject.name = 'MyApp'
+ apply from: '../node_modules/react-native-unimodules/gradle.groovy'; includeUnimodulesProjects()
  apply from: file("../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesSettingsGradle(settings)
  include ':app'
```

## expo 라이브러리 설치하기

예를 들어서 제가 사용하려고 했던 expo-screen-capture를 예로 들어보겠습니다.
아래와 같이 설치하면 됩니다.
(https://github.com/expo/expo/tree/master/packages/expo-screen-capture)

```bash
$ npx expo install expo-screen-capture
$ npx pod-install
```

그 이후에 만약 필요한 과정들이 있다면 각각의 라이브러리마다 사용법들이 문서에 적혀있으니 그대로 진행하시면 됩니다.
