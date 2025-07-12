---
title: 'allocWithZone: 의 의미'
author: yjiq150
type: post
date: 2014-07-15T13:30:08+00:00
url: /allocwithzone-의-의미/
dsq_thread_id:
  - 2845521328
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
어떤 클래스에 대해 `NSCopying` 프로토콜을 구현하다보면 `allocWithZone:` 이라는 메서드에 맞닥뜨리게된다.  
객체의 경우 힙(heap)에 할당되는데, 이 힙을 zone으로 나누고 해당 zone별로 메모리를 할당 및 관리하여 메모리 파편화를 막는데 도움을 주기위해 존재하는 메서드이다. 하지만  최근 런타임관련 알고리즘이 많이 효율적으로 변경되어서 굳이 zone을 사용하지 않아도되며, ARC로 바뀌면서 아예 zone을 사용하지 말라고 다음과같이 설명하고있다.<sup id="fnref-403-1"><a href="#fn-403-1" rel="footnote">1</a></sup>.

> You cannot use memory zones.
> 
> There is no need to use NSZone any more—they are ignored by the modern Objective-C runtime anyway. 

<li id="fn-403-1">
  <a href="https://developer.apple.com/library/mac/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html">Transitioning to ARC Release Notes</a>&#160;<a href="#fnref-403-1" rev="footnote">&#8617;</a> </fn></footnotes>