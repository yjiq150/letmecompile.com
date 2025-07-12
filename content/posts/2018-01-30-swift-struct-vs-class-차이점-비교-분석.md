---
title: Swift struct vs. class 차이점 비교 분석
author: yjiq150
type: post
date: 2018-01-30T04:06:39+00:00
url: /swift-struct-vs-class-차이점-비교-분석/
dsq_thread_id:
  - 6446890553
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/swift-closure-vs-objective-c-block/"     class="post-704"><span class="crp_title">Swift Closure vs. Objective-C Block 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/swift-closure-vs-objective-c-block/"     class="post-704"><span class="crp_title">Swift Closure vs. Objective-C Block 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - 차이

---
Swift에는 `struct`와 `class`타입이 공존하고있기 때문에 아래의 차이점을 잘 숙지하고 상황에 맞게 사용하는것이 매우 중요하다.

  * `struct` 
      * call by value (할당 또는 파라메터 전달시 value copy가 일어남)
      * stack memory 영역에 할당 (속도가 빠름) 
          * scope based lifetime: 컴파일타임에 compiler가 언제 메모리를 할당/해제할지 정확히 알고있음
          * data locality: CPU 캐시 히트율이 높음
      * 상속 불가능 (protocol은 사용 가능)
      * `NSData`로 serialize 불가능
      * `Codable` 프로토콜을 이용하여 손쉬운 `JSON` <-> `struct` 변환 가능 (Swift 4 이상)
      * 항상 새로운 변수로 copy가 일어나기때문에 multi-thread 환경에서 공유변수로 인해 문제를 일으킬 확률이 적음 <sup id="fnref-706-thread_safe"><a href="#fn-706-thread_safe" class="jetpack-footnote">1</a></sup>
  * `class` 
      * call by reference (할당 또는 파라메터 전달시 객체를 가리키고있는 메모리 주소값만 복사됨)
      * heap memory 영역에 할당 (속도가 느림) 
          * 런타임에 직접 alloc하며 reference counting을 통해 dealloc이 필요
          * memory fragmentation 등의 overhead가 존재
      * 상속 가능
      * `NSData` serialize 가능
      * `Codable` 사용 불가능
      * 런타임에 타입 캐스팅을 통해서 클래스 인스턴스에 따라 여러 동작이 가능
      * deinitializer 존재
  * 참고: class안에 struct 변수를 property로 정의하는것 가능하며, 반대로 struct의 property중 하나로 class 인스턴스 변수를 갖고있는 것도 가능하다. 이 경우 해당 struct 변수의 copy가 일어날때 class 인스턴스의 주소값만 복사된다.</p> 
  * 추가 참고자료: <sup id="fnref-706-struct_vs_class1"><a href="#fn-706-struct_vs_class1" class="jetpack-footnote">2</a></sup>

위의 technical한 차이점들을 종합하여 정리해 봤을 때 어떤상황에 어떻게 써야하는지 간단히 룰을 만들어보면 다음과 같다.

  * 상속이 필요하지 않고 모델의 사이즈가 그리 크지 않다면 struct를 사용
  * JSON의 필드와 1:1 mapping되는 간단한 모델이 필요하다면 struct를 사용 (JSON대신 다른 데이터 encoder/decoder를 구현가능하지만 Swift에서는 JSON만 제공됨)
  * 해당모델을 serialize 해서 전송하거나 파일로 저장할 일이 있다면 class 사용
  * 해당 모델이 Obj-C에서도 사용되어야 한다면 class 사용

## 특수 케이스: 클로져(Closure)와 struct

앞서 언급했듯이 `struct`, `Int` 등의 value type 변수들은 변수에 할당하거나 함수 파라메터로 전달시에 value copy가 일어난다. 하지만 Swift에서 **value type 변수가 closure에 의해 capture가 되는 경우**에는 **reference copy가 기본값**이다. 이를 피해서 명시적으로 value copy가 하고싶다면 해당 변수를 capture list<sup id="fnref-706-capture_list"><a href="#fn-706-capture_list" class="jetpack-footnote">3</a></sup>에 넣어주면 된다.

더 자세한 내용은 다음 글에 정리해 두었다.

[Swift Closure vs. Objective-C Block 비교 분석][1]

## 컬렉션(Array, Dictionary, ..) 은 어떤 타입일까?

얼핏 생각하면 `Array`, `Dictionary`같은 구조체는 다양한 기능을 포함하고 있기때문에 `class`라고 생각하기 쉽지만 Swift에서는 `struct`로 구현되어있다. 앞서 살펴본 바로는 `struct` type의 경우 대입하거나 함수 파라메터로 전달할때 call by value 형태로 copy가 되서 전달된다고 했다. 그렇다면 array를 다른 변수에 대입하거나, 함수의 파라메터로 넘길 경우에 매번 array 안에 들어있는 모든 값들을 복사해서 전달하는 overhead가 발생하진 않을까?

다행히도 이러한 overhead를 막기위해서 `Array`, `Dictionary` 같이 Swift에서 제공되는 가변길이 컬렉션들은 copy-on-write<sup id="fnref-706-copy_on_write"><a href="#fn-706-copy_on_write" class="jetpack-footnote">4</a></sup> 방식의 최적화가 적용되어있다. 실제로 array안에 저장된 가변길이 데이터들은 heap 영역 메모리 공간에 저장이 되어있으며 array 변수를 copy할 경우 해당 데이터의 메모리 주소를 참조 하고 있는 껍데기 형태의 array 변수가 새롭게 생성된다고 생각하면 된다. 실제로 데이터가 복사되지 않았기 떄문에 이렇게 껍데기만 복사하는 작업은 오버헤드가 거의 없다. 하지만 해당 array가 가진 데이터에 추가/삭제 등의 수정을 가하는 순간 실제 heap영역에 저장되어있던 데이터들에 대한 복사가 일어나고 (copy-on-write) 해당 array는 copy되어 새로 할당된 영역의 데이터에 대한 참조를 갖게 된다.

이러한 내부 동작 덕분에 array를 아무리 복사하고 파라메터로 넘기더라도 array 안에 저장된 실제 데이터는 array가 수정될때까지 복사되지 않고 오버헤드 걱정 없이 사용을 할 수 있게된다.

예시)

    let myArray = [1, 2, 3]
    
    // 단순 복사: myArray struct가 otherArray로 copy되었지만 껍데기만 복사됨
    // 메모리 사용 증가량: 거의 없음
    // -> [1, 2, 3] 데이터를 저장하고 있는 heap영역 메모리는 myArray와 otherArray가 공유해서 사용하고 있음
    let otherArray = myArray
    
    // 데이터 수정: otherArray를 수정하기 위해 실제 heap영역에서 데이터 복사가 일어나게 된다
    // 메모리 사용 증가량: 거의 2배로 늘어남
    // -> 기존에 myArray 를 위해 [1, 2, 3] 데이터에 대한 heap 영역 메모리가 할당되어있고
    // -> 신규로 otherArray를 위해 [1, 2, 3, 4] 데이터에 대한 heap 영역 메모리가 추가로 할당되었기 때문
    otherArray.append(4)
    

<li id="fn-706-thread_safe">
  <a href="https://stackoverflow.com/questions/41350772/if-arrays-are-value-types-and-therefore-get-copied-then-how-are-they-not-thread">https://stackoverflow.com/questions/41350772/if-arrays-are-value-types-and-therefore-get-copied-then-how-are-they-not-thread</a>&#160;<a href="#fnref-706-thread_safe">&#8617;</a>
</li>
<li id="fn-706-struct_vs_class1">
  <a href="https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24232845#24232845">https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24232845#24232845</a>&#160;<a href="#fnref-706-struct_vs_class1">&#8617;</a>
</li>
<li id="fn-706-capture_list">
  <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Expressions.html#//apple_ref/doc/uid/TP40014097-CH32-ID544">https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Expressions.html#//apple_ref/doc/uid/TP40014097-CH32-ID544</a>&#160;<a href="#fnref-706-capture_list">&#8617;</a>
</li>
<li id="fn-706-copy_on_write">
  <a href="https://developer.apple.com/documentation/swift/array">https://developer.apple.com/documentation/swift/array</a>&#160;<a href="#fnref-706-copy_on_write">&#8617;</a> </fn></footnotes>

 [1]: https://www.letmecompile.com/swift-closure-vs-objective-c-block/