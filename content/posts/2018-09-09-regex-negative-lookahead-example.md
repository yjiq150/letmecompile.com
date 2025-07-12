---
title: 정규식 Negative Lookahead 예제
author: yjiq150
type: post
date: 2018-09-09T14:12:49+00:00
url: /regex-negative-lookahead-example/
dsq_thread_id:
  - 6901822666
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 프로그래밍 기타

---
정규식을 사용해서 특정 조건을 만족하는 문자열을 찾되, 그중에서 제외를 하고싶은 경우(negative) 어떻게 하면 되는지 예제를 통해서 알아보자.

    String.valueOf()
    MyCustomClass.valueOf()
    Long.valueOf()
    Boolean.valueOf()
    String.valueOf()
    String.valueOf()
    String.valueOf()
    

위와 같은 문자열이 주어졌을때 `.valueOf()` 함수를 사용하는 곳을 찾고싶은데 `String.valueOf()`에 대한 검색 결과가 너무 많아서 이 케이스를 제외하고 찾고싶은 경우 Negative Lookahead를 나타내는 `(?!)` 를 이용해서 정규식을 작성하면 된다.

  * 예1) `^((?!String\.valueOf).)*\.valueOf`
  * 예2) `^((?!String|Boolean\.valueOf).)*\.valueOf` 
      * `String.valueOf()` 외에도 `Boolean.valueOf()` 까지 제외시키고 싶은 경우에 사용

정규식 작성시에는 다음 사이트에서 작업 하면 <https://regex101.com/> 바로바로 조건과 결과를 확인 할 수 있기때문에 매우 편리하다.