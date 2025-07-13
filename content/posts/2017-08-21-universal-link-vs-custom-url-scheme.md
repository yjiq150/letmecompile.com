---
title: 유니버셜링크 vs. 커스텀URL스킴 비교 분석
date: 2017-08-21T05:50:44+00:00
url: /universal-link-vs-custom-url-scheme/
dsq_thread_id:
  - 6082610800
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
iOS의 경우 기본적으로 Sandbox환경이라 다른 앱들간에 정보를 주고받는것이 간단하지 않다. 이때 정보를 주고받을 수 있는 대표적인 방법이 custom URL scheme이다. 앱의 고유한 scheme을 정의하고 이 scheme으로 시작하는 URL안에 정보를 담아서 URL을 열게되면 해당 scheme이 갖는 다른 앱을 실행하면서 정보전달을 할 수 있다. 하지만 이 방법에는 치명적인 단점이 있는데, 애플 앱스토어에서 해당 앱이 정의한 scheme의 uniqueness를 보장해주지 않기때문에 다른 앱들이 자신이 정의한 scheme을 사용하고 있을 수 있다는 점이다.

자신의 앱을 MyApp이라고 하고 이 앱이 `myapp://` 이라는 custom URL scheme을 사용한다고 가정하자. (MyApp안의 info.plist 내에 존재하는 &#8220;URL types &#8211; URL Schemes&#8221; 항목에 `myapp` 이 정의하면 됨) 방금 정의한 custom scheme`myapp`은 unique가 보장되지 않기 때문에 앱스토어에 올라온 YourApp 이라는 엉뚱한 앱이 동일하게 앱 내부에 custom scheme으로 `myapp`을 정의하는 일이 발생할 수 있다. 만약 MyApp과 YourApp이 동시에 설치되어 있는 device에서 `myapp://`으로 시작하는 URL을 open하게 되면 두 앱 중에서 누가 열릴지 알 수 없게 되어버린다. 설치 순서에 영향을 받는 것도 아니라서 특정한 규칙이 없으며, 상황에따라 두 앱중 하나가 열리게 되는 랜덤한 현상이 발생한다.

결국 유명한 앱들이 사용하는 custom scheme을 알아낸 후 마음대로 특정 앱에 정의한 후 사용하게되면 사용자가 원치 않는 엉뚱한 앱을 실행하게 만들 수 있는 보안상의 헛점이 존재하는 것이다.

이러한 보안상의 문제와 다른 app들의 custom scheme abusing을 막을 방법이 있을까 해서 Apple쪽에 컨택해 보았으나 custom scheme을 사용해서는 막을 방법이 없으니 iOS9부터 지원하는 유니버셜 링크(Universal Link)를 사용하라는 대답만 돌아왔다.

결국 어뷰징을 피하기 위해서 유니버셜 링크를 사용 할 수밖에 없기때문에 iOS9 미만에 대해서는 기존처럼 custom scheme방식을 이용하고 iOS9 이상부터는 유니버셜 링크를 사용하도록 변경하는 방법밖에는 없다. 유니버셜 링크의 경우에는 앱 번들 ID를 미리 특정 domain의 웹사이트에 등록하는 방식이라, 해당 도메인과 서버의 소유권이 있어야지만 사용할 수 있으므로 custom URL scheme처럼 abusing을 하는것이 불가능하다.

유니버셜 링크를 적용하는 방법은 다른 분들이 잘 정리해주신 여러 글들이 많으니 참고하면 된다.

  * iOS: 유니버셜 링크 적용하기 <http://ohgyun.com/706>
  * iOS: 유니버셜 링크 적용하기 앱 <http://ohgyun.com/708>
  * <https://okjungsoo.wordpress.com/2016/01/29/%EC%9C%A0%EB%8B%88%EB%B2%84%EC%85%9C-%EB%A7%81%ED%81%ACuniversal-link/>

## Universal Link 실제 동작 Case 정리

유니버셜 링크가 상황 별로 어떻게 동작하는지, 어떻게 custom scheme을 완전히 대체할 수 있는지 케이스 별로 정리해보았다. (example.com 도메인에 대해서 유니버셜 링크가 설정되어 있는 상황을 가정)

### 기본 동작

  * MyApp 내부에서 https://example.com/test 주소를 `openURL:`을 하는경우 해당주소가 Safari에서 열림
  * 다른 앱에서 http://music.line-beta.me/test 열면 Safari를 거치지 않고 바로 MyApp이 열린다.

### 동일한 scheme을 가진 앱이 2개 이상 설치된 경우

  * MyApp 내부에서 myapp://test 열었음에도 불구하고 자기 자신이 해당 URL을 핸들링하지 못하고 동일한 scheme을 가진 다른 앱이 실행되는 경우가 있다 (랜덤하게 발생).
  * 다른앱에서 myapp://test 열어도 MyApp 내부에서 해당 URL 열었을때와 동일하게 동작.

### 참고사항

  * MyApp이 보낸 푸시메시지에 https://example.com/test URL데이터가 들어있는경우, 받은 사용자가 해당 푸시메시지를 통해 앱을 실행하게 될때 `application:didFinishLaunchingWithOptions:` 이 불린다. 이때 launchOption에 들어있는 정보 기반으로 URL을 파싱하여 원하는대로 핸들링 할 수 있음
  * 유니버셜링크(Universal Link)는 iOS9 이상부터 지원되기 때문에 iOS9 미만에서 https://example.com/test 열면 그냥 일반적인 웹페이지를 열듯이 safari에서 열린다.