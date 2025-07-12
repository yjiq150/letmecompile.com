---
title: Aspect Oriented Programming in Objective-C
author: yjiq150
type: post
date: 2014-05-30T14:29:11+00:00
url: /aspect-oriented-programming-in-objective-c/
dsq_thread_id:
  - 2723981354
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
## 관점지향프로그래밍, Aspect Oriented Programming(AOP)이란?

&#8216;Aspect&#8217; 란 어떤 프로그램의 공통적인 기능(common feature)이 프로그램 전반에 걸쳐 영향을 주는 형태를 지칭한다.  
예를들어 회원 정보를 관리하는 프로그램을 만드는 경우, 실제 회원 정보를 인덱싱하는 것이 핵심적인 문제(core concern)이며, 이 부분을 수정할때에는 해당 핵심 부분만 변경하면 되기때문에 영향받는 범위가 좁아서 모듈화(modularity)이 깨지지 않는다. 하지만 이 프로그램에 로깅(logging) 기능을 추가하게되면 어떻게될까?  
로깅 기능은 DB, 인증, 등 시스템 전반에 걸쳐 적용이되어야하는 문제(cross-cutting concern)이며 따라서 로깅에 관한 코드들은 프로그램의 코드들 곳곳에 분포한다. 따라서 로깅기능을 추가하면서 기존의 객체지향프로그래밍(OOP)이나 절차지향프로그래밍(procedural programming) 방법론으로 개발할경우에는 모듈화가 깨지게 된다.  
이러한 기존의 방법론의 한계점을 해결하기위해 특정 상황에서 제한적으로 AOP를 도입하는 것을 고려해볼만 하다.

## Objective-C 에 적용하기

### 메서드 스위즐링(method swizzling)<sup id="fnref-361-nshipster"><a href="#fn-361-nshipster" rel="footnote">1</a></sup>

Objective-C의 경우 런타임의 동적인 특성 덕분에 런타임상에서 실행되는 메서드를 교체(replace) 혹은 확장(extend) 하는것이 가능한데 이를 Objective-C에서는 메서드 스위즐링이라고 부른다. 이러한 메서드 스위즐링 방법을 이용하면 AOP를 편리하게 적용할 수 있다.

### 어떤 경우에 유용한가?

iOS에서 디버깅을 위해 모든 뷰컨트롤러(viewController)들의 `viewWillAppear:` 메서드가 호출되는 시점에 로깅을 하고싶은 상황을 가정해 보자. 기존 OOP 방식대로라면 사용중인 모든 뷰컨트롤러들이 A라는 베이스 클래스로부터 상속받도록 구성하고, A 클래스의 viewWillAppear: 메서드에 로깅하는 코드를 추가해야한다. 이 경우 기존 뷰 컨트롤러들이 많을경우 클래스 상속구조를 바꾸기 위해 일일히 수정해야해서 많은 노력이 필요하다. 이런 경우에 viewWillAppear:에 대해 메서드 스위즐링을 사용하여 해당 메서드르르 원하는 메서드로 교체/또는 확장하면 훨씬 간결하게 로깅 코드를 추가할 수 있다.

메서드 스위즐링을 손쉽게 할수있도록 해주는 **Aspects(https://github.com/steipete/Aspects)** 라는 Objective-C 라이브러리를 사용하면 이러한 작업을 더 쉽게 할 수 있다. 다음 코드에서 볼 수 있듯이, 코드 한줄로 모든 UIViewController의 viewWillAppear: 함수에대해 추가적인 로깅 코드를 실행할 수 있다.<sup id="fnref-361-aspect"><a href="#fn-361-aspect" rel="footnote">2</a></sup>.

    [UIViewController aspect_hookSelector:@selector(viewWillAppear:) withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo> aspectInfo, BOOL animated) {
        NSLog(@"View Controller %@ will appear animated: %tu", aspectInfo.instance, animated);
    } error:NULL];
    

단, 메서드 스위즐링을 위해 런타임상에서 메시지 포워딩/인보케이션 등을 하는 과정에서 오버헤드가 발생할 수 있으니 자주호출되거나 성능에 민감한 부분에있어서는 사용을 주의해야 한다.

*참고: 원래 포인터 스위즐링(pointer swizzling) 이란 용어는 컴퓨터과학에서 이름 혹은 인덱스 기반으로 간접적으로 갖고있는 레퍼런스를 실제 메모리에 직접 접근가능한 포인터 레퍼런스로 변경하는것을 말한다. 대표적인 예로 직렬화(serialized)된 객체를 저장해둔 파일을 다시 메모리로 불러들일때 deserialize 하게되는데 이 과정에서 포인터 스위즐링이 일어난다. Objective-C의 메서드 스위즐링은 이러한 원래의미와는 조금 다르긴 한데, 셀렉터를 바꿔치기하여 다른 메서드가 불릴 수 있게 한다는 의미에서 비슷한 맥락으로 이해할 수 있다.<sup id="fnref-361-pointer"><a href="#fn-361-pointer" rel="footnote">3</a></sup>

<li id="fn-361-nshipster">
  http://nshipster.com/method-swizzling/&#160;<a href="#fnref-361-nshipster" rev="footnote">&#8617;</a>
</li>
<li id="fn-361-aspect">
  https://github.com/steipete/Aspects&#160;<a href="#fnref-361-aspect" rev="footnote">&#8617;</a>
</li>
<li id="fn-361-pointer">
  http://en.wikipedia.org/wiki/Pointer_swizzling&#160;<a href="#fnref-361-pointer" rev="footnote">&#8617;</a> </fn></footnotes>