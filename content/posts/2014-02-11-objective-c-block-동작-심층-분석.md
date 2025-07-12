---
title: Objective-C Block 동작 심층 분석
author: yjiq150
type: post
date: 2014-02-11T14:33:28+00:00
url: /objective-c-block-동작-심층-분석/
dsq_thread_id:
  - 2251879711
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/swift-closure-vs-objective-c-block/"     class="post-704"><span class="crp_title">Swift Closure vs. Objective-C Block 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/swift-closure-vs-objective-c-block/"     class="post-704"><span class="crp_title">Swift Closure vs. Objective-C Block 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - 코코아
  - 아이폰
  - 블럭
  - 블락
  - 변수 범위

---
블락은 애플에서 closure 개념을 도입하기위해 ANSI C 에 익스텐션 형태로 만들어진 문법이다. 따라서 C/C++/Objective-C에서 모두 사용이 가능하지만 사용법과 메모리 관리에 있어서의 사용법은 언어특성에따라 조금씩 달라진다. 여기서는 Objective-C에서의 블락에 초점을 맞춰 분석을 할 예정이다.

## 블락(Block)의 실체

블락을 선언할 경우 실제로 컴파일러에의해 생성되는 코드에서는 `__block_literal` 이라는 구조체(struct) 형태로 선언이된다. 이 구조체 안에는 `isa` 정보가 포함되어있어서 결국 Objective-C의 객체의 특성을 가지게 된다.(심지어 블락이 C/C++에서 사용되더라도 동일하게 Objective-C의 객체특성을 가진다). Obj-C의 런타임에서 `isa`에관한 세부 내용은[Objective-C 런타임 내부동작 분석][1] 글에 자세히 설명되어있다.

일반적으로 객체들은 힙(heap)에 잡히게 되지만 블락은 특이하게도 스택(stack)에 할당되는 객체이다. 이는 일반적으로 블락이 사용되는 범위가 블락이 선언되어있는 동일 범위에서 벗어나지 않기때문이다. 즉 스택에 할당하게되면 실행속도 측면에서 힙에 비해 유리하기 때문에 실행속도 최적화를위해 기본적으로 스택에 할당을 하게된다.[^1]

[<img loading="lazy" width="351" height="152" src="/uploads/2014/02/block_struct.png" alt="block_struct" class="alignnone size-full wp-image-272" />][2]

디버거로 할당된 블락 변수를 조사해보면 위의 그림과같이 구조체로 되어있음을 확인할 수 있다. 구조체 내부 변수들에서 볼 수 있듯이 해당 구조체는 블락 호출시 실행될 함수포인터를 갖고 있으며, 블락 실행시 필요한 해당 범위내(enclosing scope) 변수들도 저장하여 갖고있기때문에 해당 변수들이 저장된 크기만큼 Size가 잡혀있는 것도 볼 수 있다[^2]. 블락이 생성될때 같이 저장되는 범위내 변수들은 어떤 기준에 의해 저장되는지 더 살펴보도록 하자.

## 블락의 변수 범위

지역 스택(stack-local) 데이터가 유효한 범위는 블락을 정의하는 부분을 감싸는 바로 위 중괄호 { } 범위이다. 영어로는 same lexical scope 혹은 nearest enclosing scope 라고 표현할 수 있겠다. (lexical scope에 관해서는 [Lexical scope vs. Dynamic scope][3] 를 참조.

따라서 아래 코드와같은 패턴을 사용할경우, for문의 중괄호 범위를 벗어날 경우 해당 범위 밖에서의 블락은 유효하지 않다. 블락내부에서 사용된 변수들이 블락이 선언된 범위의 스택에 복사되어 저장되는데, 이것이 해당 중괄호 범위를 넘어가는 순간 유효하게되지 않기 때문이다.

[<img loading="lazy" width="497" height="178" src="/uploads/2014/02/block_scope.png" alt="block_scope"  class="alignnone size-full wp-image-270" />][4]

가끔 스택프레임(stack frame)에 저장되어있던 기존 내용이 다른곳에서 사용되지 않아서 덮어쓰기(overwrite)되지 않은 경우에는 블락이 운좋게 잘 실행될 경우도 있지만 일반적으로 유효성을 보장할 수 없다.

## 블락 스토리지(Block Storage)

위에서 살펴본 바와같이 블락을 만들때 실제 블락이 실행될 때를 대비하여 해당 scope에 존재하는 변수들을 저장하게되는데 이러한 변수들이 저장된 저장소를 블락스토리지(block storage)라고 부른다.

위에서 설명했듯이 블락스토리지는 최적화를 위해 처음에는 스택에 변수들을 저장한다. 하지만 스택에만 저장되어있는 경우, 해당 scope를 벗어나게되면 블락더이상 블락을 사용할 수 없게되는 문제가 발생하게된다. 이를 해결하기 위해 명시적으로 `Block_copy()` 혹은 `[blockObject copy]` 를 호출하여 블락스토리지를 힙으로 복사하여 이동시켜야만 한다. 이렇게 이동된경우, 실제로 복사된 것이므로 동일 블락 내부의 동일한 변수라도 다른 메모리 주소값을 가지게된다.

## `__block` 키워드

변수선언시 `__block` 키워드는 블락을 만들때 해당 변수를 어떤식으로 저장해두고 있을지를 정하게 되는 스토리지 타입 변경자(storage type modifier) 이다. 이 키워드의 유무에 따른 동작의 차이는 다음과 같다.

컴파일러는 블락내부에서 사용된 해당 enclosing scope에 해당하는 변수들에대해,

  * 객체일경우 자동으로 `retain`한다.
  * 객체이고 `__block` 키워드가 있을경우에는 `retain`하지 않는다.
  * 기본타입(primitive type)일 경우에는 `const` 형태로 value copy되어 전달된다.
  * 기본타입(primitive type)이고 `__block` 키워드가 있을 경우에는 reference형태로 전달된다.

아래 두 예제에서 `__block` 키워드 유무에따른 변수 값 전달 차이를 비교해보자.

[<img loading="lazy" width="264" height="148" src="/uploads/2014/02/block_value.png" alt="block_value"  class="alignnone size-full wp-image-271" />][5]

위 그림에서 블락내부 `x`는 복사되어 `const` 화된 변수이므로 할당이 불가능하다.

[<img loading="lazy" width="353" height="194" src="/uploads/2014/02/block_reference.png" alt="block_reference" class="alignnone size-full wp-image-269" />][6]

위 그림에서 블락내부 `x`는 레퍼런스를 전달받았기때문에 실제로 값을 수정가능하다.

## 순환참조(Retain cycle)주의

어떤 클래스의 인스턴스가 블락에대한 참조를 갖고있는 상황에서, 해당 블락안에서 `self`를 참조할 경우 순환참조가 발생하여 메모리 해제가 이루어 지지 않는 문제가 발생한다. 이경우 ARC가 켜져있다면 &#8220;Capturing &#8216;self&#8217; strongly in this block is likely to lead to a retain cycle&#8221; 라는 경고메시지를 출력해준다. 다음 그림을 살펴보면 `self.myBlock`이 블락객체를 참조하고 있으며, 해당 블락이 선언될때 `self`를 참조했기 때문에 블락객체 또한 `self`를 참조하게 되어 자동으로 retain 한다. 이렇게 서로 retain이 발생하기 때문에 블락 내부에서 외부 참조를 사용할 경우 주의하여야 한다.

[<img loading="lazy" width="368" height="249" src="/uploads/2014/02/retain_cycle.png" alt="retain_cycle" class="alignnone size-full wp-image-275" />][7]

[그림] 순환참조 예제 코드및 그림

ARC(Automatic Reference Counting)가 비활성화되어있을경우에는 이러한 자동 retain으로 (ARC가 활성화되어있는 경우에는 strong reference cycle 문제) 인한 순환참조를 방지하기 위해서 `self` 객체 등의 object앞에는 `__weak`(iOS 5 이상)나 `__unsafe_unretained`(iOS5 미만) 키워드를 이용하여 컴파일러에게 알려줘야한다. (이 키워드들 대신 `__block` 키워드를 사용해도 결과적는으로 동일한 동작을 하게된다.)

## 블락에 대해 retain을 하는 것이 의미가 있을까?

블락에서 사용하는 해당 범위내의 변수들은 스택에 저장되어있기때문에, 블락에대해 retain을 하더라도 블락스토리지에 들어있는 범위내 변수들은 아무런 영향을 받지 않고scope를 넘어서는 순간 해제된다. 때문에 블락에 대해서는 `copy`와 `release` 오퍼레이션만 적용가능하며 `retain` 오퍼레이션은 큰 의미가 없다.

## 블락을 프로퍼티로 사용하기

따라서 블락을 프로퍼티로 사용할경우에도 `copy` 키워드는 필수적이다.  
`@property (copy) (^void)testBlock(void)` 형태로 적어주면 된다.  
해당 블럭이 블럭 내에서 블럭이 선언될 당시의 스택프레임에 참조하는 변수들이 없다면 `copy`를 안해도 되지만 그렇지 않을경우 꼭 필요하다.

ARC환경에서 block을 사용할 경우 이미 컴파일러가 block 객체가 단순히 `retain`되면 안되고 `copy`가 되어야 한다는 사실을 알고있기때문에 block을 대입할 경우 자동으로 카피한다. 하지만 해당 변수가 해당 블락의 signature로 정확히 정의되어 있어야만 이 기능이 동작하며, 단순히 `id` 타입으로 정의되어있는 경우 제대로 작동되지 않으니 주의해야 한다.

## 블락을 이용해서 KVO 패턴을 더 편리하게 사용하기[^3]

일반적으로 KVO(Key-Value Observing)을 사용하면 `observeValueForKeyPath:ofObject:change:context:` 를 사용하게되고, 관찰하는 객체가 늘어날 수록 해당 메서드 내에 복잡한 `switch`문을 통해 일일히 어떤 객체에서 들어온 값 변화인지를 체크하는 방식으로 구현을 해야한다. 하지만 블락을 이용하여 옵저버 등록시점에 블럭을 지정하면, 값변화가 발생하면 해당 블럭을 실행되도록할 수 있어 훨씬 더 간편하게 KVO 패턴을 사용할 수 있다.

다음 두가지 구현중 마음에드는 것으로 선택해서 사용하도록 하자.

구현1: [Mike Ash&#8217;s KVO Done Right][8]

구현2: [KVO+Blocks: Block Callbacks for Cocoa Observers][9]

<div class="footnotes">
  <hr />
  
  <ol>
    <li id="fn:1">
      <p>
        https://developer.apple.com/library/ios/documentation/cocoa/conceptual/Blocks/Articles/bxOverview.html<a href="#fnref:1" rev="footnote">&#8617;</a>
      </p>
    </li>
    
    <li id="fn:2">
      <p>
        http://www.cocoawithlove.com/2009/10/how-blocks-are-implemented-and.html<a href="#fnref:2" rev="footnote">&#8617;</a>
      </p>
    </li>
    
    <li id="fn:3">
      <p>
        http://thirdcog.eu/pwcblocks/#objcblocks<a href="#fnref:3" rev="footnote">&#8617;</a>
      </p>
    </li>
  </ol>
</div>

 [1]: http://www.letmecompile.com/objective-c-%EB%9F%B0%ED%83%80%EC%9E%84runtime-%EB%82%B4%EB%B6%80-%EB%8F%99%EC%9E%91-%EB%B6%84%EC%84%9D
 [2]: /uploads/2014/02/block_struct.png
 [3]: http://en.wikipedia.org/wiki/Scope_(computer_science)
 [4]: /uploads/2014/02/block_scope.png
 [5]: /uploads/2014/02/block_value.png
 [6]: /uploads/2014/02/block_reference.png
 [7]: /uploads/2014/02/retain_cycle.png
 [8]: https://www.mikeash.com/pyblog/key-value-observing-done-right.html
 [9]: http://blog.andymatuschak.org/post/156229939/kvo-blocks-block-callbacks-for-cocoa-observers