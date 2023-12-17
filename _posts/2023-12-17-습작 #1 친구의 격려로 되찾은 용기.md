---
title: "React-Native Firebase Analytics 적용하기"
last_modified_at: 2023-12-08
categories:
  - React-Native
tags:
  - React-Native
  - Firebase
  - Analytics
---

> 소함 안드로이드 앱에서 서버 통신 소스를 떼어내면서 겸사겸사 프로젝트를 이사갔었다.  
> 기존 프로젝트에서 리액트 네이티브의 버전을 올려보려고 했지만 작업이 만만치 않아 아예 프로젝트를 통째로 옮긴것이다.  
> 그래서 이전에 연동되어 있었던 Firebase Analytics 를 다시 적용해주어야 했다.  
> Firebase 에 앱 프로젝트는 생성되어 있는 상태에서 Analytics 적용 작업을 진행하도록 하겠다.

<br>

# 1. 안드로이드 Firebase Analytics 적용하기

Firebase 콘솔에 로그인하여 프로젝트 개요 > 프로젝트 설정 화면으로 이동한다.  
일반 탭에서 스크롤을 내려보면 *SDK 안내 보기* 버튼이 있다.  
안드로이드 포커스에서 해당 버튼을 눌러보면 아래와 같은 화면이 뜬다.  

![android_google_services_json](/assets/posts/2023-12-08-01/android_google_services_json.png){: style="display:block; margin-left:auto; margin-right:auto"}

가이드에 나온대로 google-services.json 파일을 다운로드하여 [프로젝트명]/app 경로에 추가해주면 된다.  
아래와 같이 파일이 추가된 것을 확인할 수 있다.

![android_apply_google_services_json](/assets/posts/2023-12-08-01/android_apply_google_services_json.png){: style="display:block; margin-left:auto; margin-right:auto"}

다음으로 해야할 작업은 안드로이드 소스에 Firebase Analytics 연동 코드를 추가해주는 것이다.  

![android_add_sdk_1](/assets/posts/2023-12-08-01/android_add_sdk_1.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_add_sdk_2](/assets/posts/2023-12-08-01/android_add_sdk_2.png){: style="display:block; margin-left:auto; margin-right:auto"}

순서대로 먼저 android/build.gradle 파일 먼저 수정을 해준다.

![android_build_gradle](/assets/posts/2023-12-08-01/android_build_gradle.png){: style="display:block; margin-left:auto; margin-right:auto"}

다음으로 android/app/build.gradle 파일을 수정해준다.

![android_app_build_gradle_1](/assets/posts/2023-12-08-01/android_app_build_gradle_1.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_app_build_gradle_2](/assets/posts/2023-12-08-01/android_app_build_gradle_2.png){: style="display:block; margin-left:auto; margin-right:auto"}

끝으로 Android Studio 를 열고 Gradle 탭에서 app 프로젝트의 Gradle을 업데이트 해주자.

![android_studio_gradle_sync](/assets/posts/2023-12-08-01/android_studio_gradle_sync.png){: style="display:block; margin-left:auto; margin-right:auto"}

이로써 안드로이드 네이티브단 Firebase Analytics 연동 관련 작업은 마무리가 됐다.

이제 Reaat-Native 에서 Firebase Analytics 기능을 사용할 수 있도록 아래 라이브러리를 설치해주자.

```bash
yarn add @react-native-firebase/app
yarn add @react-native-firebase/analytics
```

설치가 완료된 후 앱을 빌드하여 실행시켜보자.

그러면 아래와 같이 Firebase Analytics 실시간 모니터링 화면에서 이벤트가 쌓이는 것을 볼 수 있다.

![android_complete](/assets/posts/2023-12-08-01/android_complete.png){: style="display:block; margin-left:auto; margin-right:auto"}

<br>

# 2. IOS Firebase Analytics 적용하기

Android 에서 했던 것과 마찬가지로 IOS도 진행해주면 된다.

![ios_google_service_info_plist](/assets/posts/2023-12-08-01/ios_google_service_info_plist.png){: style="display:block; margin-left:auto; margin-right:auto"}

IOS 에서는 GoogleService-Info.plist 파일을 XCode 에서 프로젝트 루트에 추가해준다.

![ios_add_google_service_info_plist](/assets/posts/2023-12-08-01/ios_add_google_service_info_plist.png){: style="display:block; margin-left:auto; margin-right:auto"}

다음으로는 firebase-ios-sdk Packages 를 이용하여 FirebaseAnalytics 를 설치해주자.

![ios_firebase_adk_add](/assets/posts/2023-12-08-01/ios_firebase_adk_add.png){: style="display:block; margin-left:auto; margin-right:auto"}

가이드대로 패키지를 검색한다.

![ios_add_analytics_packages_1](/assets/posts/2023-12-08-01/ios_add_analytics_packages_1.png){: style="display:block; margin-left:auto; margin-right:auto"}

FirebaseAnalytics를 선택한 후 설치해주자.

![ios_add_analytics_packages_2](/assets/posts/2023-12-08-01/ios_add_analytics_packages_2.png){: style="display:block; margin-left:auto; margin-right:auto"}

거의 다 왔다.

이제 IOS 에 FirebaseAnalytics 연동 소스를 추가해주자.

![ios_add_analytics_sdk_source](/assets/posts/2023-12-08-01/ios_add_analytics_sdk_source.png){: style="display:block; margin-left:auto; margin-right:auto"}

아니, 그런데 이게 왠걸.  
가이드 소스 그대로 복붙을 했는데 에러가 나는 것이 아닌가.

![ios_import_firebase_error](/assets/posts/2023-12-08-01/ios_import_firebase_error.png){: style="display:block; margin-left:auto; margin-right:auto"}

아차 ios pod 설치를 까먹었다.  
아래 명령어로 새로 추가된 React-Native 라이르러리를 ios 에 설치해주자.

```bash
cd ios
pod install
```

그런데 여전히 *import* 문에서 에러가 발생한다.  
구글링을 해보니 아래처럼 헤더파일을 *import* 시키면 된다고 한다.

![ios_import_firebase_success](/assets/posts/2023-12-08-01/ios_import_firebase_success.png){: style="display:block; margin-left:auto; margin-right:auto"}

구글링을 통해 확인한 내용대로 수정하고나니 에러가 사라졌다.  
이제 앱을 빌드하여 실행시켜보자.  
그런 다음 FirebaseAnalytics 실시간 모니터링을 통해 확인해보면 아래와 같이 IOS 이벤트가 쌓이는 것을 확인할 수 있다.

![ios_complete](/assets/posts/2023-12-08-01/ios_complete.png){: style="display:block; margin-left:auto; margin-right:auto"}

이렇게 해서 React-Native에서 Firebase Analytics 연동 작업을 완료했다.

CLEAR! 👍