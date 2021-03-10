---
layout: post
title: "ReactNative Deeplink & Universal Link"
description: "ReactNative에서 Deeplink 설정을 하면서 알게된 세팅방법과 Universal Link 세팅에 대한 개념까지 다뤄봤습니다."
author: jangwon.seo
date: 2021-03-02 22:24
tags: [reactnative, android, ios]
---

현재 [똑똑한개발자](https://toktokhan.dev/)에서는 웹앱 형태로 사이트를 구현하여 네이티브로 Wrapping 하는 식으로 웹과 네이티브를 동시에 대응하는 방향으로 개발을 하고 있습니다.

지난 주에 제가 앱에서 상세 상품 페이지를 링크로 공유했을 때, 앱이 설치된 유저와 설치되지 않은 유저를 나눠서 처리해주기 위한 로직 구현을 담당했었습니다.

# 기능

- 앱이 설치된 유저의 경우 앱 실행 후 링크 이동
- 앱이 설치되지 않은 유저의 경우 웹 다운로드 페이지로 이동

  ![redirect_page](https://github.com/TokTokHan/TokTokHan.github.io/blob/master/files/posts/2021_03/redirect_page.png?raw=true){: width="400px", marginRight="auto"}

  클라이언트 측에서 전달받은 다운로드 페이지

# 아이디어

우선 다음 같은 플로우로 처리를 하면 되지않을까 생각..

1. 공유된 링크로 웹 접근
2. 네이티브 단에서 웹뷰의 userAgent에 고유값을 추가해 해당 값으로 앱 실행여부를 파악한다.
3. 앱으로 실행하지 않았을 경우 앱을 실행시키며, 딥링크로 공유된 링크 실행
4. 딥링크 에러 또는 userAgent에서 고유값이 확인 안되었을 경우 앱에서 실행하지 않았다 판단하여 `/download` 페이지로 redirect
5. 딥링크 에러가 없다면 확인 후 url 이동

_개발하면서 알게된 부분_

**URL Schema**

⇒ `toktokhan://path`

**Universal Link**

⇒ `https://toktokhan.dev/path`

링크로 웹을 접근하는 방식이 버튼을 통한 링크 이동인지(a tag의 href), url을 통한 링크 접근인지(웹뷰로 링크 띄우기)에 따라서 인식하는게 다름.

deeplink와 universal link(app links)를 사용해서 2가지 상황에 대해서 동시에 처리해줘야함.

### 01. DeepLink 활용 [url 접근 시]

- url을 읽어 deeplink를 생성한다.
- 생성한 deeplink로 redirect 시킨다.
- timeout을 통해 redirect 후 전 페이지에 대한 이벤트 처리를 한다.

### 02. **Universal Link 활용 [링크 이동 시]**

- universal link 설정을 하면, 디바이스에 자동으로 링크를 인식해서 앱을 띄울지 선택한다.

![universal_link](https://github.com/TokTokHan/TokTokHan.github.io/blob/master/files/posts/2021_03/universal_link.png?raw=true){: width="400px", marginRight="auto"}

# 네이티브 파트

## userAgent 설정

`react-native-device-info` 라이브러리를 사용해서 userAgent를 가져와 고유값[`@toktokhan`] 를 넣어줍니다.

```jsx
...
import DeviceInfo from 'react-native-device-info';

const App = () => {
...
  return (
    <WebView
    ...
	    userAgent={`${DeviceInfo.userAgent}@toktokhan`}/>
  );
};

export default App;
```

## 딥링크 설정 [iOS]

### 01. AppDelegate.m 파일 수정

```swift
// iOS 9.x or newer
#import <React/RCTLinkingManager.h>

- (BOOL)application:(UIApplication *)application
   openURL:(NSURL *)url
   options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
  return [RCTLinkingManager application:application openURL:url options:options];
}
```

iOS 버전 9 이하의 경우 아래 코드를 사용하세요!

```swift
// iOS 8.x or older
#import <React/RCTLinkingManager.h>

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  return [RCTLinkingManager application:application openURL:url
                      sourceApplication:sourceApplication annotation:annotation];
}
```

Universal Link를 사용할 경우 아래 코드도 함께 추가해주세요.

```swift
// Universal Links
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity
 restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler
{
  return [RCTLinkingManager
            application:application
            continueUserActivity:userActivity
            restorationHandler:restorationHandler
         ];
}
```

### 02. Xcode 수정

![xcode_1](https://github.com/TokTokHan/TokTokHan.github.io/blob/master/files/posts/2021_03/xcode_1.png?raw=true)

![xcode_2](https://github.com/TokTokHan/TokTokHan.github.io/blob/master/files/posts/2021_03/xcode_2.png?raw=true)

### 03. App.js 파일 수정

```jsx
const ROOT_URL = "http://localhost:3000/" // 배포 전 수정 필요
...
const deepLinkListener = ({url}) => {
  if (url) {
    deepLink(url, 'addEventListener');
  }
};

const deepLink = (url, type) => {
  const DOMAIN = 'toktokhan.dev';
  if (url) {
    const [_, _url] = url.split('://');
    let newUrl = `${notiUrl + '/' + _url}`;
    if (_url.includes(DOMAIN)) {
      newUrl = `https://${_url}`;
    }
    webRef.current.injectJavaScript(`window.location.href = "${newUrl}";`);
  }
};

useEffect(() => {
...
  //IOS && ANDROID : 앱이 딥링크로 처음 실행될때, 앱이 열려있지 않을 때
  Linking.getInitialURL().then((url) => deepLink(url));

  //IOS : 앱이 딥링크로 처음 실행될때, 앱이 열려있지 않을 때 && 앱이 실행 중일 때
  //ANDROID : 앱이 실행 중일 때
  Linking.addEventListener('url', deepLinkListener);
	return () => {
    Linking.removeEventListener('url');
  };
}, []);
...
```

### 04. 테스트

```bash
xcrun simctl openurl booted toktokhan://product/11654
```

## 딥링크 설정 [Android]

### 01. AndroidManifest.xml 파일 수정

```xml
<activity
  android:name=".MainActivity"
  android:launchMode="singleTask">
	<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="*.toktokhan.dev" />
    <data android:scheme="https" android:host="toktokhan.dev" />
  </intent-filter>
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="toktokhan"/>
  </intent-filter>
</activity>
```

### 03. 테스트

```bash
adb shell am start -W -a android.intent.action.VIEW -d "toktokhan://product/11654" dev.toktokhan
```

# 웹 파트

### 01. \_app.tsx

```tsx
...
const checkUserAgent = () => {
  const userAgent = window?.navigator.userAgent;
  const origin = window?.location.origin;
  const pathname = window?.location.pathname;

  if (userAgent.includes('@toktokhan')) {
    if (pathname !== deepLink) {
      Router.replace(`${origin + pathname}`);
    }
  } else {
    var xhr = new XMLHttpRequest();

    xhr.onreadystatechange = () => {
      if (xhr.readyState == 4 && xhr.status == 200) {
        window.location.open(`toktokhan:/${pathname}`);
      } else {
        Router.replace(`${origin}/download`);
      }
    };

    xhr.open('head', `toktokhan:/${pathname}`);
    xhr.send(null);
  }
  DeviceStore.deepLink = pathname;
};
```

### 02. iOS Universal Link 설정 - AASA 파일 추가

`/public/.well-known/apple-app-site-association`

해당 경로에 확장자 없이 파일을 생성한 다음 아래 코드를 입력합니다.

appID 에는 `<TeamID>.<Bundle-Identifier>` 을 입력해주세요!

![apple_team_id](https://github.com/TokTokHan/TokTokHan.github.io/blob/master/files/posts/2021_03/apple_team_id.png?raw=true)

TeamID는 빨간색 박스 부분에서 확인할 수 있습니다.

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "<TeamID>.dev.toktokhan",
        "paths": ["*"]
      }
    ]
  }
}
```

### 03. Android Universal Link 설정 - assetlinks.json 파일 추가

다음 명령어를 사용하여 자바 keytool을 통해 지문 파일을 생성할 수 있습니다.

```bash
$ keytool -list -v -keystore my-release-key.keystore
```

`/public/.well-known/assetlinks.json`

해당 경로에 json 파일을 생성한 다음 아래 코드를 입력합니다.
위에서 생성한 지문 파일을 `sha256_cert_fingerprints` 에 입력해주세요.

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "dev.toktokhan",
      "sha256_cert_fingerprints": [
        "14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"
      ]
    }
  }
]
```

[[코드 생성 및 테스트]](https://developers.google.com/digital-asset-links/tools/generator) 에서 테스트를 해볼 수 있습니다!

## 참고

- [[react-native-device-info]](https://www.npmjs.com/package/react-native-device-info)
- [[react-navigation DeepLink Docs]](https://reactnavigation.org/docs/deep-linking/)
- [[react-navitve Linking Docs]](https://reactnative.dev/docs/linking)
- [[react native ios universal link]](https://rossbulat.medium.com/deep-linking-in-react-native-with-universal-links-and-url-schemes-7bc116e8ea8b)
- [[Android universal link]](https://developer.android.com/training/app-links/verify-site-associations)
