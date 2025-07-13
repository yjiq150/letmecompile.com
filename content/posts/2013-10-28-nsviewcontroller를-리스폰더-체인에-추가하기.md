---
title: NSViewController를 리스폰더 체인에 추가하기
date: 2013-10-28T11:58:02+00:00
url: /nsviewcontroller를-리스폰더-체인에-추가하기/
dsq_thread_id:
  - 1910143791
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - CocoaWithLove 번역
tags:
  - NSViewController
  - Responder chain
  - 리스폰더 체인
  - NSView

---
NSViewController가 이벤트를 수신하여 처리하기 좋은 객체인데, 이녀석은 FirstResponder가 될 수 없다보니 코코아에서 기본적으로 이벤트를 수신하는 것이 불가능하다.

NSViewController를 사용하면 NIB(혹은 xib)파일로 부터 NSView를 매우 쉽게 불러올 수 있다. 하지만 NSWindowController와는 다르게 몇가지 한계점이 있는데 이 부분을 설명하고 해결방안을 제시하고자 한다.

`NSWindowController`의 경우 `NSWindow`와 자연스레 연결하여 사용 할 수있게 해주는 다음과같은 기능들을 가지고 있다.

  * `NSWindow`객체는 `windowController`메서드를 이용하여 연결된 `NSWindowController`에 바로 접근 가능
  * First responder로서 사용자 이벤트를 수신 하여 특정 액션을 실행 가능 (responder chain의 순서는 `NSWindow`바로 다음)
  * `NSWindow`의 delegate로서 `windowWillClose:`, `windowDidBecomeMain:` 등을 수신하기 적합하고, `windowDidLoad`, `windowWillLoad` 등의 오버라이드 가능한 자체적인 메서드도 가지고 있다.

반면 `NSViewController`의 경우 `NSView`에서 `NSViewController`로의 직접적으로 접근하는 방법이 제공되지 않기때문에 기본적으로 연동 기능이 없다고 보면 된다.

## 리스폰더 체인 변경

NSViewController가 NSWindowController 수준의 기능을 갖도록 확장을 해보도록 하자. 일단 NSViewController에 연결된 view 객체를 NSView를 서브클래싱한 커스텀 클래스로 변경한다.

서브클래싱된 커스텀 클래스는 다음과 같이 viewController 연결을 위한 아울렛을 만든 후 인터페이스빌더에서 File&#8217;s Owner로 연결해주면 된다.

    IBOutlet NSViewController *viewController;
    

이제 NSViewController을 리스폰더 체인에 끼워넣기 위해 커스텀 클래스에 다음 코드를 추가하면 된다.

    - (void)setViewController:(NSViewController *)newController
    {
        if (viewController)
        {
            NSResponder *controllerNextResponder = [viewController nextResponder];
            [super setNextResponder:controllerNextResponder];
            [viewController setNextResponder:nil];
        }
    
        viewController = newController;
    
        if (newController)
        {
            NSResponder *ownNextResponder = [self nextResponder];
            [super setNextResponder: viewController];
            [viewController setNextResponder:ownNextResponder];
        }
    }
    
    - (void)setNextResponder:(NSResponder *)newNextResponder
    {
        if (viewController)
        {
            [viewController setNextResponder:newNextResponder];
            return;
        }
    
        [super setNextResponder:newNextResponder];
    }
    

인터페이스 빌더에서 아울렛을 연결해두었기 때문에 NIB파일이 로딩될때 자동으로 `setViewController:`가 호출될 것이고, 이 과정에서 `setNextResponder:`에 의해 리스폰더 체인에서 NSView 다음에 NSViewController가 추가된다.

  * 역자 주: 각각의 메서드에 브레이크 포인트를 찍어보면 알겠지만 `setViewController:`의 경우는 NIB 로딩시점에서 `[NSViewController loadView:]`가 호출될때 먼저 실행된다. `setNextReponder:` 함수는 뷰로딩 과정이 끝난 후 해당 뷰가 추가되는 시점에서 (`[NSView addView:]`가 실행될 때) 실행된다.

## 기타 고려사항

커스텀 클래스에 NSView의 메서드중 하나인 `viewDidMoveToSuperview`를 오버라이드 한 후 여기서 NSViewController쪽으로 notification을 보낼 수 있다. 이 방법을 사용하면 NSWindowDelegate가 동작하는 것과 비슷한 방식으로 NSView에 대한 delegation 방식의 처리가 가능해 진다.

### 출처 및 저작권 관련 정보

이 글의 저작권은 [CocoaWithLove.com][1]을 운영중인 Matt Gallagher 에게 있고, 저자의 동의하에 한국어로 번역되었습니다. 원본 글은 다음 링크를 통해 접근 가능합니다.

[Link to original article][2]

 [1]: http://www.cocoawithlove.com
 [2]: http://www.cocoawithlove.com/2008/07/better-integration-for-nsviewcontroller.html