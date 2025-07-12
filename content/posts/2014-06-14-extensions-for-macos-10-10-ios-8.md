---
title: 'MacOS 10.10 & iOS 8 새기능 익스텐션(Extensions) 개념 잡기'
author: yjiq150
type: post
date: 2014-06-14T06:03:45+00:00
url: /extensions-for-macos-10-10-ios-8/
dsq_thread_id:
  - 2763064176
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
기존의 OS X나 iOS에서는 custom URL Scheme을 이용하거나 custom pasteboard(번들 seed ID를 동일한경우만 가능) 등을 이용하여 어플리케이션 간에 데이터를 **&#8216;전달&#8217;**하는 것만 가능했다.

즉, A라는 앱에서 B라는 앱으로 데이터를 전달한 후 B앱으로 컨텍스트가 전환된 후 그 데이터를 받아서 추가적인 화면을 보여주거나 처리를 할 수는 있었지만, **아예 앱간 컨텍스트 전환 없이 A앱 내부에서 B앱이 가진 기능을 바로 사용하는 것이 불가능**했다.

하지만 이번에 새로운 OS에 &#8220;익스텐션(Extensions)&#8221; 이라는 이름으로 이것을 가능하게 해주는 기술을 애플에서 공개했다. 앱간에 정보를 주고받으면서 앱 to 앱으로 전환이 일어나는 것 자체가 사용자에게 있어서는 불편이었기 때문에 이러한 익스텐션 기능이 잘 적용되면 사용자 경험 자체가 많이 좋아질 것으로 예상된다.  
(사용자에게 있어서는 앱전환이 일어나지 않는것으로 보이지만, 실제로 기술적인 관점에서 봤을때 익스텐션 자체는 실행중인 앱과는 다른 독립적인 프로세스인 것에 주의)

그러다보니 iOS8과 MacOS 10.10(Yosemite)에서 새로 추가된 기능들 중 가장 많은 앱들에의해 이용될 수 있는 부분이 바로 이 익스텐션 기능인것같아서 WWDC 발표를 보면서 내용을 쭉 정리해보았다.

본 내용은 WWDC 2014 중 아래 세션 두개를 요약하면서 개념적으로 이해를 돕기위해 부분부분 내용을 추가하면서 작성되었다.

  * Session 205 Creating Extensions for iOS and OS X, part 1
  * Session 217 Creating Extensions for iOS and OS X, part 2

더 자세한 내용이 궁금해질 경우 아래 링크에서직접 WWDC 영상을 참고 바람(애플 개발자 계정 필요).

<https://developer.apple.com/videos/wwdc/2014/>

## 익스텐션이란?

익스텐션, 정확히 말해 익스텐션 컨테이너(Extension Container)는 다음과 같은 특징을 지닌다.

  * 앱이 아니다.
  * 애플 framework 코드를 통해서만 접근된다.
  * App to app IPC (Inter-Process Communication)가 아니다.
  * 빌드될때 추가적인 타겟을 통해 따로 빌드되며 설치될때는 앱과 같이 설치, 삭제될때도 앱과 함께 삭제된다. (바이너리 자체도 앱과 독립적이다)
  * 실행시에도 앱과는 완전히 다른 독립 프로세스(process)로 실행되기때문에  완전히 다른 주소공간(Isolated address space)를 가지게 된다.

한마디로 **익스텐션**은 애플 프레임웍을 통해서 호출되는 기능들의 집합이라고 표현할 수 있다. 익스텐션의 기능들은 애플프레임웍를 사용해서만 실행 가능하며. 호스트앱(host app)이 다이렉트로  익스텐션을 호출하지 못한다. 실행이 애플 프레임웍에의해 된다는 사실을 제외하면, 앱과 거의 동일한 형태를 가진다. 즉, 유저 인터페이스도 가질 수 있고, 이를위한 ViewController, 리소스 등을 모두 사용가능하다.

Pre-release 된 애플의 문서<sup id="fnref-381-pre_release"><a href="#fn-381-pre_release" rel="footnote">1</a></sup>에 나와있는 다음 다이어그램을 보면 좀더 이해가 잘 될것이다.  
<img loading="lazy" width="720" height="323" src="/uploads/2014/06/bundle.png" alt="bundle" class="alignnone size-full wp-image-379" /> 

좀더 이해를 돕기위해 사진앨범 앱과 카메라 필터앱의 예를 들어보자. 기존에는 사진앨범 앱에서 카메라 필터앱으로 사진데이터를 넘긴 후, 필터 앱에서 편집된 사진을 다시 사진앨범 앱으로 넘겨야 했다.  
하지만 카메라 필터앱에서 익스텐션 형태로 추가적인 구현을 할 경우 사진앨범 앱에서 사용자가 사진에대한 custom action 메뉴를 선택한 경우 해당 카메라 필터 익스텐션을 선택할 수 있게 된다. 이때 이 필터 익스텐션을 선택하면 사진앨범 앱에서 같은 앱을 사용하고있는것처럼 바로 필터를 적용하는 UI가 나타날 것이고 (카메라 필터 앱으로 전환이 일어나지 않는다) 여기서 바로 필터 적용 및 저장이 가능해지는 것이다.

## 애플에서 허용한 익스텐션 종류

현재 이러한 익스텐션 방식으로 구현할 수 있도록 다음 세가지를 애플에서 제공해준다. 더 자세한 내용은 여기<sup id="fnref-381-ext_type"><a href="#fn-381-ext_type" rel="footnote">2</a></sup> 을 참조.

  * 투데이 익스텐션(Today Extension), 
    말그대로 알림센터(notification center) 내부에 투데이 부분에 앱에서 원하는 기능을 위젯형태로 추가 할 수 있게되었다.

  * 공유 익스텐션(Share Extension)
    
    사용자가 무언가를 SNS로 공유하려할 때 경우, 트위터나 페이스북이 아닌 경우라도 바로 포스팅할 수 있도록 해준다.

  * 사진 편집 익스텐션(Photo Editing Extension)
    
    사진 편집을 바로 할수있게해줌.

  * 액션 익스텐션(Action Extension)
    
    이미지, 사운드 등 알려진 형식의 데이터나, 새롭게 정의한 커스텀 타입의 특정데이터들을 이용할 경우, 해당 형식의 데이터를 인식하고 처리할 수 있는 익스텐션이 OS에 등록되어 있다면 사용자의 액션에 의해 해당 익스텐션을 사용할 수 있게 된다.

## 앱 &#8211; 익스텐션 간 코드를 공유하는 방법

동일한 기능을 가진 부분을 프레임웍으로 추출하여 앱, 익스텐션 두곳에서 공유하여 사용하는 형태로 개발하는 것이 효율적이다.

### Mac OS 에서의 프레임웍 공유

Mac OS의 경우 이미 예전부터 런타임에 동적 링크(dynamic link)가 가능한 framework을 만들어서 사용하는것이 허용되어왔다. 프레임웍의 경우 공유여부에 따라 다음과 같이 두가지로 나누어질 수 있다. <sup id="fnref-381-mac_ref"><a href="#fn-381-mac_ref" rel="footnote">3</a></sup>.

  * public framework 
      * 설치시 `/Library/Frameworks` or `~/Library/Frameworks` 로 복사.
      * 다른 앱에서도 해당 framework를 동적으로 링크할 수 있음.
      * 여러 앱에서 동일한 framework을 로딩하더라도 처음 한번만 메모리상에 로드되고 해당 부분을 공유하여 사용하기때문에 메모리 공간을 아낄 수 있음.
      * 여러 앱에서 공유되고있는 framework이 업데이트되어 version이 바뀔경우, 특정 어플리케이션에서는 문제가발생 할 수 있기때문에 버전관리 및 호환성 유지에 매우 주의해야한다.
  * private framework 
      * 앱 번들 내부에 포함됨(embeded in an app bundle)
      * 해당 프레임웍은 앱 내부에서만 사용가능하며, 다른 앱에서 접근하는 것이 불가능해짐.
      * 하나의 앱에서만 사용되므로 framework vesioning을 신경쓸 필요가 없음

### iOS 에서의 프레임웍 공유

iOS8 이전에는 Xcode에서 직접적으로 framework을 만들 수 있도록 템플릿을 지원해 주지 않았다. 대신 직접 shellscript등을 이용하여 프레임웍 디렉토리를 구성하는 방식으로 framework을 만들 수 있는 방법들은 있었지만 (예: Facebook iOS framework), 이 또한 보안(security) 문제로 정적(static link) 방식으로만 한정되어있었다. 동적 링크가 가능한 경우 어플리케이션 실행중에

> 참고: iOS 앱의 앱번들을 열어보면 framework들이 컴파일타임에 이미 정적으로 링크되었기때문에 frameworks 디렉토리가 존재하지 않고, 하나의 실행 바이너리(executable binary)로 모두 링크되어 통합된 것을 확인 할 수 있다. 

하지만 iOS8 부터 Xcode에서 프레임웍을 만들 수 있는 템플릿이 추가되었고, 동적 링킹(dynamic linking)이 가능하도록 변경되었다. 하지만 앱번들에 프레임웍이 embeded되는 형태로만 허용되며 framework을 암호화(encrypt) 하는 과정을 거친다. 때문에 해당 앱과 익스텐션에서만 링크가 가능하며, Mac OS에서 허용된 것처럼 다른 앱들간에 프레임웍을 공유하는 것은 여전히 불가능하다.

<!-- 동적 링킹을 하는 framework을 사용할 경우 Minmum deployment target은 현재 iOS 8 이상인 것으로 알려져 있고, 이 WWDC에서 부분은 바뀔 수도있다고 언급 된걸로 보아 확정된 이야기는 아닌것 같다. -->

### 기타: API 제한사항

대부분의 API가 사용가능하지만 익스텐션에서 사용불가능한 몇몇 예외가 존재하니, 사용전에 해당 API에 사용불가능 macro로 (`NS_EXTENSION_UNAVAILABLE_IOS`) 표기되어있는지 확인할 필요가 있다.

앱과 익스텐션은 다른 독립 프로세스기때문에 익스텐션에서 앱의`UIApplication`에 접근하는것이 당연히 불가능하다. 따라서 그런 구조적으로 사용이 불가능할 수 밖에 없는 API들은 다음 예제처럼 매크로가 미리 추가되어있다.

    + (UIApplication *)sharedApplication 
    NS_EXTENSION_UNAVAILABLE_IOS("Use view controller based solutions where appropriate instead.");
    

## 앱 &#8211; 익스텐션 간 데이터 공유 방법

앱과 익스텐션간에 데이터를 공유하기 위하여 앱그룹(App Group) 이라는 새로운 컨셉을 도입하였다.  
앱과 익스텐션은 동일한 앱그룹(App Group)에 속하며 동일 앱그룹 내에서는 저장소와 일반적인 데이터들을  공유할 수 있게 허용된다.

<img loading="lazy" width="204" height="215" src="/uploads/2014/06/appgroup.png" alt="appgroup" class="alignnone size-full wp-image-376" /> 

  * 앱과 익스텐션은 독립적인 프로세스로 실행되기때문에  두개 모두 동시에 실행중일 수 있다.  
     ex) 앱에서 백그라운드 작업이 진행되고있는 상황에서 익스텐션 기능을 실행하는 경우  
    따라서 공유되는 데이터를 읽고쓰는 경우 동기화(synchronization) 가 필수적이다.

<img loading="lazy" width="744" height="276" src="/uploads/2014/06/process.png" alt="process" class="alignnone size-medium wp-image-380" /> 

### 동기화 방법

`CoreData` 또는 `sqlite`를 이용하면 이미 기본적인 동기화 기능이 제공되며  
좀 더 커스터마이즈 하고싶은경우, 일반적인 inter-process (앱-익스텐션간) 동기화 방법을 제공하는 `NSFileCoordination` 을 이용하여 동기화 를 하면된다.

### NSUserDefaults 공유

앱과 익스텐션은 기본적으로 다른 NSUserDefaults 도메인을 가지기 때문에, 앱에서 쓰던 설정값을 익스텐션에서는 사용하지 못한다.

설정값을 앱/익스텐션간 공유하려면 다음과 같이 명시적으로 공유를 위한 shared domain을 생성해서 사용해야한다.

    NSUserDefaults *sharedDefaults = [[NSUserDefaults alloc] initWithSuiteName:@“com.mycompany.MyGreatApp.shared”];
    

### 키체인(Keychain) 공유

**번들 시드 ID** (bundle seed ID, bundle ID prefix 또는 teamID 라고도 함)만 같으면 앱/앱 간에도 키체인을 공유하는 것이 기존에도 가능했다. 이는 익스텐션에도 동일하게 적용되어, 해당 키체인 액세스 그룹에 bundle seed ID 같게 되어있으면 익스텐션/앱 사이에도 손쉽게 키체인을 공유할 수 있다.

참고: 아래 full ID에서 EF42RFSZ가 번들 시드 ID 값이다.

> EF42RFSZ.com.example.myApp 

## 익스텐션을 위한 info.plist 설정

`info.plist`파일에 현재 익스텐션이 어떤 형태의 데이터를 다룰 수 있는지, 동일한 데이터들이 여러개 선택되었을때 최대 몇개까지 처리가능한지 등의 정보를 미리 정의해 두어야 한다. `NSExtension` 속성값 아래에 `PrincipalClass`, `RoleType` 등을 적절히 정의하도록 하자

롤 타입에 `NSExtenstionServiceRoleTypeEditor` 값을 설정한경우 경우 사용자가 문서 편집모드에 들어갔을때 활성화 된다.

<img loading="lazy" width="809" height="194" class="alignnone size-full wp-image-375" alt="plist" src="/uploads/2014/06/plist.png" /> 

## 결론

앱 익스텐션으로 인해 사용자 편의성이 많이 좋아질 것으로 기대되며, 특정 분야에 최적화 된 기능을 가진 앱들이 익스텐션을 통해 시장 점유율을 더 확고히 다질 수 있게 될 것다. 아직 프리릴리즈 단계이니 미리 준비해서 iOS8 릴리즈와 함께 재빨리 익스텐션이 포함된 앱을 릴리즈하면 Feature App으로 선정될 수 있는 좋은 기회이기도 하다.

<li id="fn-381-pre_release">
  https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW2&#160;<a href="#fnref-381-pre_release" rev="footnote">&#8617;</a>
</li>
<li id="fn-381-ext_type">
  https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/NotificationCenter.html#//apple_ref/doc/uid/TP40014214-CH11-SW1&#160;<a href="#fnref-381-ext_type" rev="footnote">&#8617;</a>
</li>
<li id="fn-381-mac_ref">
  https://developer.apple.com/library/mac/documentation/macosx/conceptual/BPFrameworks/Tasks/InstallingFrameworks.html#//apple_ref/doc/uid/20002261-BBCCFBJA&#160;<a href="#fnref-381-mac_ref" rev="footnote">&#8617;</a> </fn></footnotes>