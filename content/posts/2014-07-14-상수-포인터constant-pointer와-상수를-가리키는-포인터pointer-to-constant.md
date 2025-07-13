---
title: 상수 포인터(constant pointer)와 상수를 가리키는 포인터(pointer to constant)
date: 2014-07-14T09:19:19+00:00
url: /상수-포인터constant-pointer와-상수를-가리키는-포인터pointer-to-constant/
dsq_thread_id:
  - 2842010246
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - 프로그래밍 기타

---
C문법에서 `const` 키워드가 포인터에대해 사용될때 키워드의 위치에따라 의미가 매우 달라진다. 상수포인터와, 상수를 가리키는 포인터 변수 두가지가있으니 주의해서 사용해야한다.

## 상수 포인터(constant pointer)

  * 문법: 
         int * const myValue;
        

  * 포인터 변수가 갖고있는 주소 값을 변경 불가능</p> 
  * 포인터 변수가 가리키는 주소에 존재하는 값을 변경하는 것은 가능 
  * Objective-C에서 키값을 정의하는데 많이 사용되는 패턴이다.
    
    예): NSString *const kCustomKey = @“myKey”;

## 상수를 가리키는 포인터(pointer to constant)

  * 문법:
    
        const int *  myValue;
        

  * 포인터 변수가 갖고있는 값은 마음대로 변경 가능

  * 포인터 변수는 가리키고 있는 주소에 존재하는 값을 변경하는것이 불가능하다.