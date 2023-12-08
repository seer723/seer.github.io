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

> 소함 안드로이드 앱에서 서버 통신 소스를 떼어내면서 프로젝트를 이사갔었다.   
> 그래서 이전에 연동되어 있었던 Firebase Analytics 를 다시 적용해주어야 한다.  
> Firebase에 앱 프로젝트는 생성되어 있는 상태에서 진행하도록 하겠다.

<br>

# 1. 안드로이드 Firebase Analytics 적용하기

![android_google_services_json](/assets/posts/2023-12-08-01/android_google_services_json.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_apply_google_services_json](/assets/posts/2023-12-08-01/android_apply_google_services_json.png){: style="display:block; margin-left:auto; margin-right:auto"}


![android_add_sdk_1](/assets/posts/2023-12-08-01/android_add_sdk_1.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_add_sdk_2](/assets/posts/2023-12-08-01/android_add_sdk_2.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_build_gradle](/assets/posts/2023-12-08-01/android_build_gradle.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_app_build_gradle_1](/assets/posts/2023-12-08-01/android_app_build_gradle_1.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_app_build_gradle_2](/assets/posts/2023-12-08-01/android_app_build_gradle_2.png){: style="display:block; margin-left:auto; margin-right:auto"}

```bash
yarn add @react-native-firebase/app
yarn add @react-native-firebase/analytics
```

![android_studio_gradle_sync](/assets/posts/2023-12-08-01/android_studio_gradle_sync.png){: style="display:block; margin-left:auto; margin-right:auto"}

![android_complete](/assets/posts/2023-12-08-01/android_complete.png){: style="display:block; margin-left:auto; margin-right:auto"}

<br>

# 2. IOS Firebase Analytics 적용하기

![ios_google_service_info_plist](/assets/posts/2023-12-08-01/ios_google_service_info_plist.png){: style="display:block; margin-left:auto; margin-right:auto"}

![ios_add_google_service_info_plist](/assets/posts/2023-12-08-01/ios_add_google_service_info_plist.jpg){: style="display:block; margin-left:auto; margin-right:auto"}

![ios_firebase_adk_add](/assets/posts/2023-12-08-01/ios_firebase_adk_add.png){: style="display:block; margin-left:auto; margin-right:auto"}

![ios_add_analytics_packages_1](/assets/posts/2023-12-08-01/ios_add_analytics_packages_1.jpg){: style="display:block; margin-left:auto; margin-right:auto"}

![ios_add_analytics_packages_2](/assets/posts/2023-12-08-01/ios_add_analytics_packages_2.jpg){: style="display:block; margin-left:auto; margin-right:auto"}

![ios_add_analytics_sdk_source](/assets/posts/2023-12-08-01/ios_add_analytics_sdk_source.jpg){: style="display:block; margin-left:auto; margin-right:auto"}

![ios_import_firebase_error](/assets/posts/2023-12-08-01/ios_import_firebase_error.jpg){: style="display:block; margin-left:auto; margin-right:auto"}

```bash
cd ios
pod install
```

![ios_import_firebase_success](/assets/posts/2023-12-08-01/ios_import_firebase_success.jpg){: style="display:block; margin-left:auto; margin-right:auto"}

![ios_complete](/assets/posts/2023-12-08-01/ios_complete.png){: style="display:block; margin-left:auto; margin-right:auto"}