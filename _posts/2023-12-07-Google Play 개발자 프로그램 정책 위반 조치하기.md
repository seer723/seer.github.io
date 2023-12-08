---
title: "Google Play 개발자 프로그램 정책 위반 조치하기"
last_modified_at: 2023-12-07
categories:
  - Android
tags:
  - Android
  - GooglePlay
  - PlayConsole
---

> 소함 안드로이드 앱 관리자 페이지에 Google Play 개발자 정책을 위반했다는 경고 메시지가 떴다.  
> 그래서 해결방법을 고객센터에 문의해보니 위반사항이 조치된 버전의 앱을 배포하면 된다는 답변을 받았다.  
> 앱 정책 위반 경고 메시지가 떴을 땐 당황하지 말고 수정된 버전을 배포해주자.

<br>

# 1. 구글 Play Console 에서 정책 위반 경고 알림 확인

이제 앱 창업에 대한 마음을 정리했기 때문에 소함 앱내 서버 기능을 제거하려한다.  
그래서 앱에 추가되어 있는 서버 통신 기능들을 모두 떼어냈다.  
그러고나니 명상화면과 달력화면만 남았다.  
깔끔하고 좋았다.  
제자리를 찾은 듯한 이 느낌은 뭐지.  
그렇게 앱을 빌드하고 평소처럼 Google Play Console 에 접속해서 앱을 배포하려했다.  
그런데 이게 왠걸.  
Google Play 정책을 위반했다는 경고 알림이 떡하니 있었다.  

![policy_violation_alert_notification](/assets/posts/2023-12-07-01/policy_violation_alert_notification.png)

<br>

# 2. 개발자 프로그램 정책 위반 내용 확인

정책 위반 사항을 보니 안드로이드 앱의 SDK API 수준을 33으로 타겟팅해야한다는 내용이었다.   

![policy_violation_warning_contents](/assets/posts/2023-12-07-01/policy_violation_warning_contents.png)

확인해보니까 예전에 비공개테스트에 등록해놓은 앱의 타겟 SDK 버전이 31로 되어있었다.

![reason_for_policy_violation](/assets/posts/2023-12-07-01/reason_for_policy_violation.png)

그런데 어떻게 조치해야하는지 방법을 모르는 것이 문제였다.  
그래서 고객센터에 문의했다.  
Google Play Console 에서 상담원과의 채팅기능을 제공하는데 이 기능이 꽤 좋았다.  
답변을 받아보니 타겟 SDK 버전이 33으로 수정된 앱을 비공개 테스트 트랙에 등록해주면 해결된다고 하였다.  
그래서 바로 수정작업 돌입.

<br>

# 3. 정책 위반 사항 조치 및 해결 완료 메시지 확인

android > build.gradle 파일을 열고 targetSdkVersion 을 33으로 바꿔주었다.

![android_build_gradle_target_sdkverion](/assets/posts/2023-12-07-01/
android_build_gradle_target_sdkverion.png){: style="display:block; margin-left:auto; margin-right:auto"}

그런 다음 앱을 다시 빌드하여 비공개 테스트 트랙에 앱을 등록하고나서 타겟 SDK 를 확인해보니 33으로 잘 들어가있었다.

![policy_violation_resolved_contents](/assets/posts/2023-12-07-01/policy_violation_resolved_contents.png)

잠시 뒤 앱 위반 경고 알림은 사라지고 대신 정책 위반이 해결되었다는 알림이 떴다.  
정책 위반으로 경고를 받은 경우에는 해당 이슈를 조치하고 해결 알림이 뜨는지까지 확인해야 정상적으로 완료된것이다.

![policy_violation_resolved_message](/assets/posts/2023-12-07-01/policy_violation_resolved_message.png)

정책 위반 경고 알림을 발견하고 당황한 순간부터 정책 위반 해결 알림을 받기까지 아는 내용은 거의 없었지만 고객센터에 물어보고 구글링하면서 이슈를 해결할 수 있었다.  
역시 부딪히면서 배우고 하면 할 수 있다는 생각이 중요한 것 같다.  
CLEAR! 👍