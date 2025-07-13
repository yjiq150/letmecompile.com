---
title: Objective-C 런타임(runtime) 내부 동작 분석
date: 2013-08-17T02:44:54+00:00
url: /objective-c-런타임runtime-내부-동작-분석/
dsq_thread_id:
  - 1611958733
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - Objective-C 런타임 동작 방식
  - NSObject 상속 이유
  - 메타클래스
  - 오브젝티브C
  - objective-C와 C++ 코드를 섞어 쓸 수 있는 이유
  - unrecognized selector
  - Objective-C 메시지 엑셉션
  - objc_msgSend
  - objective-c isa 변수
  - isa
  - sel
  - imp
  - class cache
  - 클래스 캐시
  - Obj-C
  - Objective-C 런타임 동작 원리
  - 카테고리
  - 카테고리 멤버변수 추가
  - associated reference
  - Associated object
  - 셀렉터
  - Objective-C 셀렉터
  - 메시지 포워딩
  - Objective-C 메시지 포워딩
  - Objective-C 런타임
  - Objective-C runtime
  - Objective-C

---
맥의 코코아(Cocoa)나 iOS의 코코아터치(CocoaTouch) 프레임워크를 다루다보면 Objective-C 런타임(runtime)과 항상 맞닥뜨리게 된다. 입문자들의 경우에는 프레임워크를 이용해서 잘 동작하는 Objective-C 코드를 작성하는데만 급급하지만, 해당 객체가 응답할 수 없는 잘못된 메시지를 보내서 런타임 에러가 나는 것 등의 여러가지 예외 상황을 겪게되면 점점 Objective-C 런타임이 어떻게 동작하는지 궁금해지게 된다. 필자도 이것이 궁금해져서 구글링을 하다가 설명이 잘되어있는 Colin Wheeler의 포스팅[^1]을 발견했다. 본 글의 내용들은 이 포스팅을 번역하여 작성되었고 내용상의 설명순서는 좀더 이해하기 쉽게 재배열 하였다. 부분적으로 추가 설명이 더 필요한 부문은 애플의 Objective-C Runtime Programming Guide[^2]도 참조했다.

## Objective-C 런타임 개념

런타임이 있다는 이야기는 Objective-C가 동적인 언어(dynamic language)라는 것이다. 정적인 언어(static language)들의 경우 컴파일/링크 시점에서 이미 어떤 코드가 어떻게 실행될지 모든것이 결정된다. 하지만 런타임을 지닌 동적인 언어의 경우 이러한 결정을 실제 그 코드가 실행되는 순간까지 미룰 수가 있는 특징을 지닌다. 이러한 특징을 덕분에 실제 코드가 실행중인 런타임 상황에서 Objective-C 런타임에의해 원하는 객체로 메시지를 리다이렉트 한다던지, 메서드 자체를 바꿔치기 한다던지 등의 유연한 동작이 가능해진다.

### 메시지 전송 예시

Objective-C 문법의 특징인 대괄호 [ ] 사이에 쓰여지는 코드에 의해 Objective-C의 메세지(message)를 생성한다. 다음 예시를 보면 메시지를 수신할 리시버와 보낼 메시지 형태로 코드가 구성되고 이 코드가 컴파일러에 의해 C코드로 컨버젼되며 결국 이를 통해 리시버(타겟) 인스턴스에 대해 메소드 호출이 일어남을 알 수 있다.

메시지를 보내는 Objective-C 코드:

<pre><code>[receiver message]
</code></pre>

컴파일러에 의해 컨버젼된 코드:

<pre><code>objc_msgSend(receiver, selector)
</code></pre>

인자(argument)가 있을경우:

<pre><code>objc_msgSend(receiver, selector, arg1, arg2, ...)
</code></pre>

이러한 메세지 방식은 언뜻 보면 C의 함수 호출과 비슷해 보인다. 하지만 Obj-C에서는 타겟 객체에게 메시지를 보낸다고 해서 그 메시지에 포함된 명령이 100% 수행되는 것은 아니다. 타겟 객체는 메시지가 누구로 부터 왔는지 확인 해서 다른 메소드을 실행할지 혹은 다른 타겟 객체에게 메시지를 포워딩 할지를 결정하게 된다.

## Objective-C 런타임 소스 분석

Objective-C 런타임은 애플사에의해 오픈소스로 공개되어있고 다음 사이트에서 내려받을 수 있다. [http://opensource.apple.com][1]. 현재 마운틴라이언에 내장된 최신버전의 런타임은 [objc4-532.2][2] 이다.  
위의 런타임 소스코드를 열어보면 대부분 C와 어셈블리로 짜여져 있으며, C 언어에 객체지향기능을 사용할 수 있도록 추가해서 Objective-C의 동작을 가능하게 해주는 라이브러리이다.  
실제 소스코드를 열어보고 어떤식으로 구현이 되어있는지 살펴보도록 하자.

### 런타임 클래스 정보 `objc.h`

런타임 라이브러리의 내부에는 다음과 같은 Objective-C의 클래스(`objc_class`)와 클래스로부터 생성된 객체(`objc_object`)를 표현하기 위한 C의 구조체 코드가 있다.

<pre><code>typedef struct objc_class *Class;
typedef struct objc_object {
    Class isa;
} *id;
</code></pre>

모든 `objc_object`들은 `isa`라고 정의된 클래스 변수를 갖고있다.  
Objective-C 런타임은 이 `isa` 포인터를 이용하여 해당 객체가 어떤 클래스인지, 그리고 이 객체가 명령 메시지를 받았을때 셀렉터에 응답을 하는지를 확인한다. 마지막으로 이러한 `objc_object`들은 `id` 포인터로 `typedef` 되게 된다. 따라서 Objective-C 코드에서 이 `id` 포인터를 이용하게되면 해당 포인터가 지칭하고 있는 객체가 어떤 클래스인지, 어떤 메서드를 갖고있는지 등의 확인이 가능한 것이다.

### 셀렉터 `objc.h`

셀렉터는 Obj-C의 메서드를 가리키는 구조체로 C언어로 다음과 같이 정의된다.

<pre><code>typedef struct objc_selector  *SEL; 
</code></pre>

실제 사용은 다음과 같이 할 수 있다.

<pre><code>SEL aSel = @selector(sampleMethod);
</code></pre>

### 메서드 호출 `objc.h`

이제 Objective-C의 메서드가 어떤식으로 컴파일되어 동작하는지 확인하기 위해 다음 구문을 보자.

<pre><code>typedef id (*IMP)(id self,SEL _cmd,...); 
</code></pre>

`IMP`는 &#8220;**id를 리턴**하고 **타겟**, **셀렉터 명령**, 및 **추가 파라메터**들을 인자(argument)로 받는 함수 포인터&#8221;를 `typedef`한 것이다. (위 `typedef` 구문이 잘 이해가지 않는다면 분들은 C에서 함수포인터를 typedef 하는 방법에 대해 찾아 볼 것을 권장한다.)  
결국 함수포인터 IMP는 Objective-C 메서드가 컴파일되어 생긴 C 함수를 포인팅 하게 되고, 이를 통해서 원하는 Objective-C 메서드에 대응하는 C 함수 호출이 일어난다.

### 상속 및 추가 정보 `runtime.h`

다음과 같은 `NSObject`를 상속받은 빈 클래스가 있다고 가정하자.

<pre><code>@interface EmptyClass : NSObject {

}
@end
</code></pre>

빈 클래스임에도 불구하고 실제로 컴파일 시에 다음 코드를 통해 추가적인 정보가 자동으로 `EmptyClass`에 포함되게 된다. `objc_object`에서와 마찬가지로 `objc_class`에서도 `isa` 클래스 변수를 통해 해당 클래스의 정보에 접근이 가능하다. 또한 `super_class` 포인터를 통해서 상속받은 `NSObject`에 접근이 가능하며, `objc_protocol_list`, `objc_method_list` 등의 정보를 통해 실제 클래스의 세부적인 정보들을 알 수 있게 된다. 결국 Objective-C 컴파일러가 컴파일시 자동으로 삽입한 이러한 추가정보들은 Objective-C 런타임 환경에서의 동적인 동작을 위해 사용되는 것이다.

<pre><code>struct objc_class {
    Class isa;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif
} 
</code></pre>

## 메타클래스(MetaClass): 클래스 정의 자체도 객체다.

앞서 소스코드에서 보았듯이 Objective-C 런타임에서 Objective-C의 &#8220;클래스&#8221; 정의를 컴파일 할 때 메타클래스(MetaClass)를 생성해서 처리를하게된다. 다시말해 Objective-C 인스턴스 객체 뿐만 아니라 &#8220;클래스&#8221; 정의 자체도 이미 객체화 되어 있다는 것이다. 이런 특징 덕분에 `[NSObject alloc]` 같이 &#8220;클래스&#8221;에 `alloc` 메세지를 보내는 것이 가능한 것이다. 이러한 &#8220;클래스&#8221; 객체는 메타클래스의 인스턴스여야 하고, 이 메타클래스 또한 최상위(root) 메타 클래스의 인스턴스여야 한다.

NSObject로 부터 상속받은 클래스의 경우, 이 클래스는 superclass로 NSObject를 가리킨다. 하지만 모든 메타클래스들은 최상위 메타 클래스를 자신의 superclass로 가리키며 응답가능한 클래스 메서드들에 대한 정보를 갖고있다.  
그래서 실제로 `[NSObject alloc]` 코드가 실행되는 과정을 살펴보면, 먼저 objc_msgSend()를 통해 메시지가 `NSObject`의 메타클래스로 전달되고 `alloc`이라는 클래스 메서드가 있는지 확인한 후에 실행을 하게 된다.

두가지 메서드가 호출되는 방법을 비교하면 다음과 같다.

  * 인스턴스 메서드(- 로 시작하는 메서드) 호출 -> 일반 인스턴스 객체로 메시지 전달
  * 클래스 메서드(+ 로 시작하는 메서드) -> 메타클래스 객체로 메시지 전달

## `NSObject` 클래스를 상속받는 이유

Objective-C 프로그래밍을 하다보면 아무 생각 없이 `NSObject` 클래스를 상속받아 사용하고있는 자신을 발견하게 된다. 결국 `NSObject`를 상속받음으로 해서 이제까지 설명한 Objective-C 런타임의 동작을 위한 추가 정보들이 자동으로 모든 클래스들에 포함되어 컴파일 되는것이다. 이러한 정보들 덕분에 우리가 객체를 생성할 때, 런타임이 원하는 구조에 맞는 형태로 메모리에 객체가 손쉽게 생성되는 것이다.

## 클래스 캐쉬( `objc_cache *cache` )

Objective-C 런타임이 isa포인터를 이용하여 해당 객체를 조사하면 해당 객체가 가진 수많은 메서드들을 확인하고 호출 가능하다. 하지만 일반적으로 모든 메서드들이 골고루 호출되는 것은 아니기 때문에 클래스 디스패치 테이블(class dispatch table)을 매번 검색하여 호출하는 것은 성능상 좋지않다.

때문에 성능향상을 위해 일단 메서드가 한번 호출되면, 해당 메서드의 셀렉터가 클래스 내부에 존재하는 캐쉬에 저장되게 된다. 따라서 objc_msgSend()에 의해 전송된 메시지를 받으면 먼저 캐쉬에서 확인을 한 후에 해당되는 것이 없을경우 클래스 디스패치 테이블을 검색하게 된다.

이 사실을 염두해 두고 `MyObject *obj = [[MyObject alloc] init];` 코드가 실행될때 내부적으로 어떻게 동작하는지 상세히 살펴보도록 하자.

<pre><code>@implementation MyObject
-(id)init {
    if(self = [super init]){
        [self setVarA:@"blah"];
    }
    return self;
}
@end
</code></pre>

  1. `[MyObject alloc]` 메시지를 수신하면 `MyObject` 클래스 자체에는 `+alloc`이 구현되어있지 않기때문에 `superclass` 포인터를 따라 `NSObject`로 올라간다.
  2. `NSObject`는 `+alloc`에 응답을 한다(`NSObject` 클래스의 캐쉬에 `+alloc`이 저장된다). `+alloc` 메서드가 호출되면서 리시버 클래스(즉, `MyObject`)의 사이즈에 맞게 메모리 블락을 할당하고 `isa`포인터를 `MyObject` 클래스로 설정한다.
  3. 이제까지는 클래스 메시지가 어떻게 보내지는지 다루었으니 클래스 인스턴스에 메시지가 전달되었을때의 동작을 살펴보자. `MyObject`는 `-init` 에 응답하므로, 이 메서드가 호출되며 `MyObject` 클래스 캐쉬에 저장된다.
  4. `self = [super init]` 호출됨: `super`는 `superclass` 포인터를 참조하는 예약어(magic word)이다. 이를 통해서 NSObject에도 `-init` 메시지가 전달된다.

## `+alloc`, `-init` 이 항상 해당 클래스와 동일한 객체를 리턴할까?

<pre><code>int main (int argc, const char * argv[]) {
    NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];

 id obj1 = [NSMutableArray alloc];
 id obj2 = [[NSMutableArray alloc] init];

 id obj3 = [NSArray alloc];
 id obj4 = [[NSArray alloc] initWithObjects:@"Hello",nil];

 NSLog(@"obj1 class is %@",NSStringFromClass([obj1 class]));
 NSLog(@"obj2 class is %@",NSStringFromClass([obj2 class]));

 NSLog(@"obj3 class is %@",NSStringFromClass([obj3 class]));
 NSLog(@"obj4 class is %@",NSStringFromClass([obj4 class]));

 id obj5 = [MyObject alloc];
 id obj6 = [[MyObject alloc] init];

 NSLog(@"obj5 class is %@",NSStringFromClass([obj5 class]));
 NSLog(@"obj6 class is %@",NSStringFromClass([obj6 class]));

 [pool drain];
    return 0;
}
</code></pre>

아마도 다음과 같이 출력되리라 예상하겠지만,

<pre><code>obj1 class is NSMutableArray
obj2 class is NSMutableArray 
obj3 class is NSArray
obj4 class is NSArray
obj5 class is MyObject
obj6 class is MyObject
</code></pre>

실제 출력 결과는 다음과 같다.

<pre><code>obj1 class is __NSPlaceholderArray
obj2 class is NSCFArray
obj3 class is __NSPlaceholderArray
obj4 class is NSCFArray
obj5 class is MyObject
obj6 class is MyObject
</code></pre>

즉, Objective-C에서는 `+alloc`이 특정 클래스의 객체를 리턴하고, 이 객체에 `-init`메시지를 보내서 또 다른 객체를 리턴해주는 경우도 존재한다는 의미이다.

## objc_msgSend()의 동작

다음 코드는

<pre><code>[self printMessageWithString:@"Hello World!"];
</code></pre>

컴파일되면서 실제로는 다음과 같은 코드를 생성한다.

<pre><code>objc_msgSend(self,@selector(printMessageWithString:),@"Hello World!");
</code></pre>

이 코드로 부터 타겟 객체의 isa 포인터를 이용하여 해당 메시지에 대한 셀렉터 응답여부를 체크하고 실행하게 된다. 실제로 `objc_msgSend()`는 아무 값도 리턴하지 않지만, 이 메시지에 의해 실행되는 메서드가 결과를 리턴 하기 때문에 `objc_msgSend()`가 리턴한 것처럼 보인다.

타겟 객체에 메시지가 보내졌을때의 동작은 다음과 같은 순서로 일어난다.

  1. 무시되는 셀렉터(ex: garbage collection이 활성화 되있을 경우 `retain`, `release` 등)인지 체크
  2. 타겟이 `nil`인지 체크. (덕분에 Objective-C의 경우 `nil`에게 메시지를 보내더라도 아무 문제가 되지 않는다.)
  3. 클래스 캐쉬에서 IMP를 발견한 경우 해당 포인터를 따라 해당 함수로 jump.
  4. 클래스 캐쉬에 없다면 클래스 디스패치 테이블 검색하여 IMP를 찾고 해당 함수로 jump.
  5. 위의 두가지 경우에서 모두 IMP가 발견되지 않은 경우 포워딩 메커니즘으로 jump 하게된다.

## Objective-C 메서드의 실체

Objective-C 메서드는 컴파일러에 의해 C 함수로 트랜스폼 된다. 예를들어 아래와 같은 이름을 가진 메서드는

<pre><code>-(int) doComputeWithNum:(int)aNum 
</code></pre>

다음과 같은 C함수로 컨버전 된다.

<pre><code>int aClass_doComputeWithNum(aClass *self,SEL _cmd,int aNum) 
</code></pre>

그리고 Objective-C 런타임은 함수포인터를 이용해 해당 함수를 호출한다.

일반적으로 이렇게 생성된 C 함수에 직접적인 접근이 불가능하지만, 코코아에서 다음과같은 방법을 제공해 준다.

<pre><code>//declare C function pointer
int (computeNum *)(id,SEL,int);

//methodForSelector is COCOA & not ObjC Runtime
//gets the same function pointer objc_msgSend gets
computeNum = (int (*)(id,SEL,int))[target methodForSelector:@selector(doComputeWithNum:)];

//execute the C function pointer returned by the runtime
computeNum(obj,@selector(doComputeWithNum:),aNum); 
</code></pre>

위의 방법을 통해서 런타임에 의한 정해진 동작을 우회하여 해당 C 함수에 대한 직접적인 호출이 가능하며, 실제로 이 방식은 Objective-C 런타임이 `objc_msgSend()`를 통해서 메서드를 호출하는 방식과 동일하다.

## Objective-C 메시지 포워딩(Message Forwarding)

Objective-C에서는 해당 객체가 메시지에 응답 가능한지 모르는 상태에서 메시지를 보내는 것이 아무런 문제가 되지 않도록 설계되어있다. 애플사의 문서에는 Objective-C가 언어차원에서 지원하지 않는 다중 상속을 시뮬레이트하기 위한 것이 그 이유중 하나라고 나와있다. 또한 이러한 특징을 이용하여 메시지를 처리하는 객체/클래스들을 숨겨서 디자인을 추상화 하고싶은 경우에도 사용이 가능하다.

실제 객체에 메시지가 전송되었을때 일어나는 일들을 순서대로 살펴보도록 하자.

  1. 런타임은 클래스 캐쉬와 디스패치 테이블을 찾아보고 해당 메서드가 응답하는지 확인한다.
  2. 해당메서드에 응답하는 것을 찾지 못한경우 런타임은 `+ (BOOL) resolveInstanceMethod:(SEL)aSEL` 를 해당 클래스에 대해 호출한다. 이를 통해서 사용자가 메소드가 없을경우의 동작을 사용자가 직접 정의할 수 있는 기회가 주어진다.
    
    <pre><code>void fooMethod(id obj, SEL _cmd)
 {
  NSLog(@"Doing Foo");
 }

 + (BOOL)resolveInstanceMethod:(SEL)aSEL
 {
     if(aSEL == @selector(doFoo:)){
         class_addMethod([self class],aSEL,(IMP)fooMethod,"v@:");
         return YES;
     }
     return [super resolveInstanceMethod];
 }
</code></pre>

`class_addMethod()`의 마지막 인자로 들어가있는 `"v@:"`가 메서드의 시그너쳐(signature)를 나타낸다(v는 `void` 리턴값,`@`는 객체 인자). Objective-C Runtime Programming Guide[^2]의 Type Encodings section 에 이 값을 어떻게 사용되는지 명시되어 있다.

  3. `resolveInstanceMethod:`가 `NO`를 리턴할경우, 런타임은 `- (id)forwardingTargetForSelector:(SEL)aSelector`을 호출하게 된다. 이를 통해서 다른 객체가 해당 메시지에 응답할 기회를 다시한번 주게되며 다음과 같이 구현한다. (이 함수에서 `self`를 리턴할 경우 무한루프에 빠지게되므로 주의할것. )
    
    <pre><code>- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(mysteriousMethod:)){
        return alternateObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}
</code></pre>

이 메서드는 속도가 느린(expensive call) `forwardInvocation:`메서드로 메시지가 리디렉션 되기 전의 마지막 처리 기회이고, 일반적인 포워딩보다 10배정도 빠르게 동작하기 때문에 기본적인 프록시(proxying) 상황에서 유용하다.

  4. 위 함수에서도 `nil`값이 리턴되면, 런타임은 최종적으로 `- (void)forwardInvocation:(NSInvocation *)anInvocation`을 호출한다. `NSInvocation`은 Objective-C 메시지가 객체형태로 표현된 것이라고 생각하면 된다. 따라서 일단 `NSInvocation`이 생성되면, 메시지의 타겟, 셀렉터, 인수 등을 마음대로 변경이 가능해진다.
    
    <pre><code>-(void)forwardInvocation:(NSInvocation *)invocation
{
    SEL invSEL = invocation.selector;

    if([altObject respondsToSelector:invSEL]) {
        [invocation invokeWithTarget:altObject];
    } else {
        [self doesNotRecognizeSelector:invSEL];
    }
}
</code></pre>

위의 함수를 오버라이드하지 않을경우 디폴트 동작으로 `-doesNotRecognizeSelector:`가 호출된다.

## Objective-C 연관 객체(Associated Object)

Objective-C의 카테고리(category)기능은 손이 많이가는 상속과정 없이도 기존에 존재하는 클래스에 추가로 메서드를 추가할 수 있게 해준다. 하지만 멤버 변수를 추가 할 수 없다는 것이 불편한 점이었다. 하지만 Mac OS X 10.6 스노우 레오파드(Snow Leopard), iOS 4.0 부터 Associated References 라는 기능이 추가되면서 이를 해결 할 수 있게 되었다(카테고리에 멤버 변수를 get/set하는 것 처럼 시뮬레이트하는 메서드를 추가하는 방식). 이미 존재하는 클래스에 멤버 변수를 추가하기 위한 다음 코드를 보자.

<pre><code>#import &lt; Cocoa/Cocoa.h&gt; //Cocoa
#include &lt;objc/runtime.h&gt; //objc runtime api's

@interface NSView (CustomAdditions)
@property(retain) NSImage *customImage;
@end

@implementation NSView (CustomAdditions)

static char img_key; //has a unique address (identifier)

-(NSImage *)customImage
{
    return objc_getAssociatedObject(self,&img_key);
}

-(void)setCustomImage:(NSImage *)image
{
    objc_setAssociatedObject(self,&img_key,image,
                             OBJC_ASSOCIATION_RETAIN);
}

@end
</code></pre>

위 코드는 기본적으로 기존 카테고리(category)를 선언할때와 동일하다. 이에 추가적으로 멤버 변수 처럼 동작을 시뮬레이트 하기 위해 `@property`가 정의되었고, `static` 키값 변수를 통해서 associated 될 객체들을 get/set 하도록 되어있다.

`objc_getAssociatedObject()` 함수 사용을 위해 `#include <objc/runtime.h>`을 추가해줬고

`runtime.h`에 보면 set을 할때 필요한 옵션상수값들이 명시되어 있으니 해당 옵션 값을 `@property`정의할때 사용 했던 것과 일치시켜 사용하면 된다.

<pre><code>objc_setAssociatedObject(). 
/* Associated Object support. */

/* objc_setAssociatedObject() options */
enum {
    OBJC_ASSOCIATION_ASSIGN = 0,
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1,
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,
    OBJC_ASSOCIATION_RETAIN = 01401,
    OBJC_ASSOCIATION_COPY = 01403
}; 
</code></pre>

## Objective-C를 C/C++ 코드와 섞어쓸 수 있는 이유?

Objective-C는 C에 대해 OOP(object oriented programming)과 런타임 모델을 추가한 확장판이라고 생각할 수 있다. 위에서 본 것 처럼, Objective-C 코드는 컴파일러에의해 결국 C 코드로 컨버전 되는데, 이것이 바로 C/C++과 Objective-C를 섞어서 코딩해도 아무 무리없이 같이 잘 동작하는 이유이다.

<div class="footnotes">
  <hr />
  
  <ol>
    <li id="fn:1">
      <p>
        http://cocoasamurai.blogspot.kr/2010/01/understanding-objective-c-runtime.html<a href="#fnref:1" rev="footnote">&#8617;</a>
      </p>
    </li>
    <li id="fn:2">
      <p>
        http://developer.apple.com/mac/library/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html<a href="#fnref:2" rev="footnote">&#8617;</a>
      </p>
    </li>
  </ol>
</div>

 [1]: http://opensource.apple.com/
 [2]: http://opensource.apple.com/source/objc4/objc4-532.2/
