---
title: NSView debug drawing
author: yjiq150
type: post
date: 2014-05-10T10:24:24+00:00
url: /nsview-debug-drawing/
dsq_thread_id:
  - 2674515284
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
tags:
  - 디버그 드로잉
  - debug draw
  - nsview layout

---
# NSView debug drawing

NSView의 레이아웃 계층(layout hierachy)이 어떻게 구성되어있는지 비주얼라이즈해서 보고싶은경우 `NSShowAllViews` Preference값을 `YES`로 셋팅해준 후 어플리케이션을 다시 실행하면 된다.  
터미널의 커맨드라인에서 다음 명령을 실행하면 해당 번들ID를 가진 앱의 설정값을 변경 가능하다.

    defaults write com.your.apps.bundlename NSShowAllViews YES
    

[<img loading="lazy" width="777" height="522" src="http://www.letmecompile.com/wp/wp-content/uploads/2014/05/nsview_debug_draw.png" alt="nsview_debug_draw"  class="alignnone size-full wp-image-340" />][1]

Constraint기반의 레이아웃이 어떻게 적용되고있는지는 `NSConstraintBasedLayoutVisualizeMutuallyExclusiveConstraints`을 이용하여 볼 수 있다.

    defaults write com.your.apps.bundlename NSConstraintBasedLayoutVisualizeMutuallyExclusiveConstraints YES
    

반대로 다시 끄고싶으면 값을 NO로 바꾸어 재실행한다.

### Preference 값을 지정하는 다른 방법

앞서 소개한것 처럼 커맨드라인에서 `NSUserDefaults` 키값을 강제로 바꿔줄 수도있지만, Xcode상에서 이 값을 scheme에 설정하여 실행할때마다 오버라이드하여 사용하는 방법 또한 편리하다.  
Xcode의 Edit Scheme 메뉴로 들어가서 &#8216;Arguments&#8217; 탭의 Arguments Passed On Launch 항목에 아래 스크린샷처럼 `-NSShowAllViews YES` 라고 추가해 주면 해당 스킴에 의해 빌드되서 실행되는 어플리케이션의 경우 해당 값이 항상 설정되어 실행된다.

[<img loading="lazy" width="690" height="453" src="http://www.letmecompile.com/wp/wp-content/uploads/2014/05/argument.png" alt="argument" class="alignnone size-full wp-image-344" />][2]

 [1]: http://www.letmecompile.com/wp/wp-content/uploads/2014/05/nsview_debug_draw.png
 [2]: http://www.letmecompile.com/wp/wp-content/uploads/2014/05/argument.png