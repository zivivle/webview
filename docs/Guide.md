## 가이드 목차

- 기본 인라인 HTML
- 기본 URL 소스
- 로컬 HTML 파일 로드
- 탐색 상태 제어
- 파일 업로드 지원 추가
- 다중 파일 업로드
- 파일 다운로드 지원 추가
- JS와 네이티브 간의 통신
- 사용자 정의 헤더, 세션 및 쿠키 사용
- 페이지 탐색 제스처 및 버튼 지원

## 기본 인라인 HTML

WebView를 사용하는 가장 간단한 방법은 표시할 HTML을 그대로 전달하는 것이다. html 소스를 설정할 때는 originWhiteList 속성을 ['*']로 설정해야한다.

```js
import React, { Component } from "react";
import { WebView } from "react-native-webview";

class MyInlineWeb extends Component {
  render() {
    return (
      <WebView
        originWhitelist={["*"]}
        source={{ html: "<h1>This is a static HTML source!</h1>" }}
      />
    );
  }
}
```

-> 새로운 정적인 html 소스를 전달하면 WebView가 다시 렌더링된다

## 기본 URL 소스

WebView에서 가장 일반적인으로 사용되는 방법이다.

```js
import React, { Component } from "react";
import { WebView } from "react-native-webview";

class MyWeb extends Component {
  render() {
    return (
      <WebView
        source={{
          uri: "https://reactnative.dev/",
        }}
      />
    );
  }
}
```

## 로컬 HTML 파일 로드하기

때때로 앱과 함께 HTML 파일을 번들로 제공하고 WebView에 해당 HTML을 로드하고 싶을 수 있다.

iOS와 Windows에서는 아래와 같이 HTML 파일을 다른 자산처럼 가져올 수 있다.

```js
import React, { Component } from "react";
import { WebView } from "react-native-webview";

const myHtmlFile = require("./my-asset-folder/local-site.html");

class MyWeb extends Component {
  render() {
    return <WebView source={myHtmlFile} />;
  }
}
```

하지만 Android에서는 HTML 파일을 Android 프로젝트의 디렉터리에 넣어야 한다. 예를 들어, local-site.html이 HTML 파일이고 이 파일을 WebView에 로드하려면 해당 파일을 프로젝트의 Android 디렉터리 (your-project/android/app/src/main/assets/)로 이동해야한다.

```js
import React, { Component } from "react";
import { WebView } from "react-native-webview";

class MyWeb extends Component {
  render() {
    return (
      <WebView source={{ uri: "file:///android_asset/local-site.html" }} />
    );
  }
}
```

## 탐색 상태 변경 제어

때로는 사용자가 WebView에서 링크를 클릭했을 때 다른 작업을 수행하거나 해당 링크로 이동하는 대신 다른 처리를 하고 싶을 수 있다.

- `onNavigationStateChange` 함수를 사용하여 이 작업을 수행하는 예시 코드

```js
import React, { Component } from "react";
import { WebView } from "react-native-webview";

class MyWeb extends Component {
  webview = null;

  render() {
    return (
      <WebView
        ref={(ref) => (this.webview = ref)}
        source={{ uri: "https://reactnative.dev/" }}
        onNavigationStateChange={this.handleWebViewNavigationStateChange}
      />
    );
  }

  handleWebViewNavigationStateChange = (newNavState) => {
    // newNavState looks something like this:
    // {
    //   url?: string;
    //   title?: string;
    //   loading?: boolean;
    //   canGoBack?: boolean;
    //   canGoForward?: boolean;
    // }
    const { url } = newNavState;
    if (!url) return;

    // handle certain doctypes
    if (url.includes(".pdf")) {
      this.webview.stopLoading();
      // open a modal with the PDF viewer
    }

    // one way to handle a successful form submit is via query strings
    if (url.includes("?message=success")) {
      this.webview.stopLoading();
      // maybe close this view?
    }

    // one way to handle errors is via query string
    if (url.includes("?errors=true")) {
      this.webview.stopLoading();
    }

    // redirect somewhere else
    if (url.includes("google.com")) {
      const newURL = "https://reactnative.dev/";
      const redirectTo = 'window.location = "' + newURL + '"';
      this.webview.injectJavaScript(redirectTo);
    }
  };
}
```

## 파일 업로드 지원 추가

### iOS

: iOS에서는 ios/[project]/Info.plist 파일에 다음과 같은 권한을 지정하면 된다.

#### 사진 촬영

```js
<key>NSCameraUsageDescription</key>
<string>Take pictures for certain activities</string>
```

#### 갤러리 선택

```js
<key>NSPhotoLibraryUsageDescription</key>
<string>Select pictures for certain activities</string>
```

#### 동영상 녹화

```js
<key>NSMicrophoneUsageDescription</key>
<string>Need microphone access for recording videos</string>
```

### Android

: Android에서는 AndroidManifest.xml 파일에 권한을 추가하면 된다

```js
<manifest ...>
  ......

  <!-- this is required only for Android 4.1-5.1 (api 16-22)  -->
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

  ......
</manifest>
```

#### Android에서 카메라 옵션 활성화

: 파일 입력에서 accept 속성을 통해 이미지나 비디오를 요구할 경우, WebView는 사용자가 카메라를 이용해 사진이나 비디오를 촬영할 수 있는 옵션을 제공한다.

```js
<queries>
  <intent>
    <action android:name="android.media.action.IMAGE_CAPTURE" />
  </intent>
</queries>
```

또한 카메라가 capture 속성으로 지정된 경우, AndroidManifest.xml 파일에 다음을 추가해야 일부 Android 버전 및 기기에서 카메라가 일관되게 작동한다.

#### 파일 업로드 지원 확인 (static isFileUploadSupported() 사용)

```js
import { WebView } from "react-native-webview";

WebView.isFileUploadSupported().then((res) => {
  if (res === true) {
    // file upload is supported
  } else {
    // not file upload support
  }
});
```
