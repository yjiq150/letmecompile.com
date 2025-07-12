---
title: Xcode Debugging tips
author: yjiq150
type: post
date: 2014-05-20T15:00:27+00:00
url: /xcode-debugging-tips/
dsq_thread_id:
  - 2704053497
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - 엑스코드 디버깅
  - debugging tutorial
  - debug tip
  - 디버그 팁
  - 맥 디버그
  - 아이폰 디버그

---
엑스코드(Xcode)를 사용하면서 여러가지 유용하게 사용해왔던 기능들을 간단히 정리해보았다.  
본 글에서는 잘 사용하지 않아서 모르고있지만 유용하게 사용될 수 있는 기능위주로 설명을 할 것이고 실제 디버거를 이용한 프로그램에대한 디버깅 방법은 다음 글에 더 자세히 적어두었으니 참조하도록 하자.  
<http://www.letmecompile.com/xcode-lldb-디버깅-테크닉/>

## Xcode 디버깅 콘솔에 색상 적용하기

네트워크 어플리케이션을 개발하다보면 다양한 로그들이 여러 스레드에서 복합적으로 출력되어 한눈에 알아보기가 쉽지 않다. 이때 CocoaLumberjack과 XcodeColors 플러그인의 조합으로 콘솔화면에 로깅 타입에따라서 색상 적용이 가능해서 매우 편리하다.

다음 링크를 참고해서 설정하면 된다.

<https://github.com/CocoaLumberjack/CocoaLumberjack/wiki/XcodeColors>

해당 프로젝트 Scheme의 Environment variable에 XcodeColors = YES 를 설정해주는것도 잊지 말자.

## Preprocessor Macro가 프로세스 완료된 소스코드 보기

`#define` 혹은 `#if` 등의 `#`으로 시작하는 프리프로세서 매크로(preprocessor macro)들은 컴파일 직전에 process 되어서 컴파일을 위한 최종 소스코드를 생성하게 된다. 복잡한 프리프로세서 매크로를 사용하다보면 이렇게 preprocess가 끝난 최종 소스코드를 보고 실제 어떻게 처리되었는지 싶을 경우가 있다.  
이때는 아래 그림과 같이 Xcode 편집창의 좌상단 메뉴를 클릭해서 Preprocess 메뉴를 선택하면 된다.  
[<img loading="lazy" width="470" height="434" src="http://www.letmecompile.com/wp/wp-content/uploads/2014/05/menu.png" alt="menu" class="alignnone size-full wp-image-353" />][1]

libobjcext에 포함되어있는 매크로인 `@strongify`와 `@weakify` 의 경우 block 프로그래밍 할때 자주 쓰이는데, 실제로 어떤 코드로 프로세스되는지 한번 살펴보았다.

[<img loading="lazy" width="1893" height="875" src="http://www.letmecompile.com/wp/wp-content/uploads/2014/05/preprocessed.png" alt="preprocessed"  class="alignnone size-full wp-image-352" />][2]

## 프로그램 실행시 파라메터/환경변수 설정하기

Xcode에서 해당프로젝트의 scheme 편집메뉴에 들어가게되면 Run 항목에 대해 **&#8220;Arguments Passed On Launch&#8221;**와 **&#8220;Environment Variables&#8221;** 두개의 항목이 있다.

[<img loading="lazy" width="690" height="453" src="http://www.letmecompile.com/wp/wp-content/uploads/2014/05/argument.png" alt="argument" class="alignnone size-full wp-image-344" />][3]

### Arguments Passed On Launch

이부분은 해당 어플리케이션에대해 NSUserDefault 값을 실행시에 오버라이드 할때 사용된다.  
시스템상에 있는 값들을 오버라이드하여 사용하는것도 가능하지만, 사용자 정의 키값을 앱 실행시 기본값으로 지정하여 실행할 수 있기 때문에 디버깅에 매우 유용하다.

다음은 코어데이터 관련해서 디버깅에 도움을 주는 시스템에 미리 정의된 인자값들이다.

  * `-com.apple.CoreData.MigrationDebug 1` 
    데이터모델이 변경되어 마이그레이션이 일어날때 관련 로그를 추가적으로 출력.

  * `-com.apple.CoreData.SQLDebug 3`
    
    sqlite가 backing store로 지정되어있을때 SQLDebug 에 로그레벨을 지정하면, 조정해서 이에대한 로그를 얼마나 자세히 출력할지 설정이 가능하다.

  * `-com.apple.CoreData.SyntaxColoredLogging YES`
    
    로깅에 컬러를 적용.

### Environment Variables

다음 설명하는 환경변수들은 Mac OS X 어플리케이션에만 적용되니 주의 할것.

  * `NSDragManagerLogLevel = 1~6`  
    드래그&드롭 액션시 출력되는 로그레벨을 1~6까지 조정가능하다.</p> 
  * `NSBindingDebugLogLevel = 1`  
    바인딩 설정이 잘못된경우 undefined key exception 말고도 추가적인 로그를 출력해줌.

  * `OBJC_HELP` 실행시 OBJC_ 로 시작하는 환경변수들의 목록을 콘솔에 모두 출력해준다. 출력한 후 적당히 원하는 변수들만 찾아서 사용하면 되고, 필자에게 유용하게 느껴졌던 몇가지 만 아래에 소개해본다.

  * `OBJC_PRINT_REPLACED_METHODS` 중복된 카테고리 명이 있는지 체크하여 경고를 해준다.
  * `OBJC_PRINT_VTABLE_IMAGES` 오버라이드된 메서드를 보여주는 vtable을 출력한다.

그외에 더 많은 팁들이 아래 링크에 소개되어있으니 참고하자.  
<http://nshipster.com/launch-arguments-and-environment-variables/>

## 기타 디버깅 팁 모음

남들이 모르는 더 많은 팁을 원한다면 무려 제목이 Debugging Magic인 다음 두 글을 참고하도록 하자.  
두 문서에 공통적인 내용들도 많지만 OS dependent 한 내용들도 많이 있으니 특정방법은 Mac에선 되고 iOS에서는 안되는 것들이 있으니 주의해서 살펴보아야 한다.

[Technical Note TN2124: Mac OS X Debugging Magic][4]

[Technical Note TN2239: iOS Debugging Magic][5]

 [1]: http://www.letmecompile.com/wp/wp-content/uploads/2014/05/menu.png
 [2]: http://www.letmecompile.com/wp/wp-content/uploads/2014/05/preprocessed.png
 [3]: http://www.letmecompile.com/wp/wp-content/uploads/2014/05/argument.png
 [4]: https://developer.apple.com/library/mac/technotes/tn2124/_index.html
 [5]: https://developer.apple.com/library/ios/technotes/tn2239/_index.html