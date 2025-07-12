---
title: C auto, static, extern 키워드 의미
author: yjiq150
type: post
date: 2014-02-20T13:02:04+00:00
url: /c-auto-static-extern-키워드-의미/
dsq_thread_id:
  - 2293581711
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - 프로그래밍 기타

---
C에서 변수는 값이 저장되는 메모리 영역이나 변수의 유효범위에 따라 구분되는데 `auto`, `static`, `extern` 이 세가지 키워드(keyword)를 이용하여 구분을 할 수 있다.

변수의 유효범위는 크게 지역변수(local variable)와 전역변수(global variable)로 나뉘며, 실제 프로그램 실행시에 변수가 저장되는 메모리상의 위치에따라 높은 주소값부터 거꾸로 사용하여 내려오는 스택(stack)과, 낮은 주소값부터 올라가면서 사용하는 정적데이터영역(.data, .bss)과 힙(heap)으로 구분된다<sup id="fnref-1"><a href="#fn-1" rel="footnote">1</a></sup>. 최적화 관점에서 살펴보면, 스택의 경우 해당 범위(scope)에서 자주 액세스 되며 범위가 끝나면 없어지는 임시변수들이 저장되는데, 스택포인터를 순차적으로 이동해가면서 할당되기 때문에 할당속도가 빠르며, 해당 변수들이 자주사용되어서 CPU 캐시의 힛트율이 높은 경우 접근 속도도 더 빨라질 가능성이 있다. 하지만 범위가 끝나면 없어지기때문에 계속적으로 값을 유지하는 것이 불가능하다. 이러한 문제를 해결하기위해서는 정적인 변수들은 정적 데이터 영역에 저장하여 계속 유지되며, 동적인 변수들은 힙에 변수를 할당하고 저장해야한다. 힙은 비순차적으로 메모리 할당/해제가 계속 일어나는 동적인 특성을 갖고있다. 때문에 스택처럼 메모리 할당을 순차적으로 하기가 힘들어서 정적인 방식에 비해 할당 속도가 느린 특성을 갖는다.

이제 각각의 키워드에 따라 해당 변수의 유효범위와 저장되는 영역이 어떻게 결정되는지를 케이스별로 살펴보도록 하자.

  1. `auto` (자동범위변수) 
      * 일반적인 지역 변수 형태로 블럭 안에서만 유효하며 블럭의 실행이 끝나면 소멸
      * 스택에 메모리 할당
      * 일반적으로 C에서 `auto` 키워드는 생략되어있음. 즉 아무 표시하지 않은 변수는 `auto`와 같은 의미.
      * C++에서 auto 키워드를 사용할 경우 &#8220;자동 타입 추론&#8221;이라는 완전히 다른 의미를 가지게되므로 주의할것.
  2. `static` (정적변수) 
      * 블럭 안에서만 유효한 값을 가지지만 자동변수와 같이 없어지지 않고 블럭으로 다시  
        돌아왔을 때 이전 값을 다시 이용 가능(스택이 아닌 정적데이터 영역에 저장됨)
      * 초기화를 생략하면 0으로 자동 초기화
      * 정적 데이터영역에 할당
      * `static`은 **사용되는 위치에 따라 의미가 달라지니 주의해서 사용**할것. 
          * 내부정적변수 : 함수 내부의 변수에 static이 사용된경우, 해당 변수는 함수 내부에서만 사용이 가능. 하지만, 프로그램이 실행되는 동안 계속 값이 유지됨
          * 외부정적변수 : 함수 외부의 전역범위의 변수에 static이 사용된경우, 해당 파일 내부에서는 전역변수처럼 사용되지만, 아닌 다른 소스파일에서는 참조할 수 없음  
            (예: 보통 하나의 example.c 파일 내부에서만 사용하기위한 전역변수로 사용됨. 다른 파일이 example.h를 include하더라도 해당 `static`으로 선언된 전역변수는 보이지 않아 접근 불가)
    
     **static_example.h** 
    
          extern int global_value; // 오류
          ...
          ...
        
    
     **static_example.c** 
    
          #include "static_example.h"
        
          static int global_value = 4; // static_example.c 파일 내부에서만 유효
        
          function test() {
             ...
          }
        
    
     **myfunction.c** 
    
          #include "static_example.h"
        
          function printGlobalVariableInOtherFile() {
             printf("value:%d", global_value); // 오류(변수 접근 불가)
          }
        

  3. `extern` (외부변수) 
      * 함수 밖의 전역 범위에 선언되며, 프로그램 전체에서 유효하고 다른 파일에서도 참조 가능
      * 초기화를 생략하면 0으로 자동 초기화
      * 정적 데이터영역에 할당
      * extern 변수는 편리하지만 남발하면 프로그램을 복잡하게 하고 나중에 유지보수가  
        힘들기 때문에 사용을 최소화 하는 것이 바람직함</p> 
      * 외부변수 정의하는 방법은 아래 extern_example 예제를 참조하고 실제 이를 다른파일에서 사용하는 예시는 `myfunction.c` 를 보면된다.
    
     **extern_example.h** 
    
         // 이 선언을 통해 global_value 라는 변수가 메모리상에 존재하며 사용될 수 있음을 알린다.
          extern int global_value; 
          ...
          ...
        
    
     **extern_example.c** 
    
          #include "extern_example.h"
        
          int global_value = 4; // 다른 파일에서도 참조가능한 전역변수
        
          function test() {
             ...
          }
        
    
     **myfunction.c** 
    
          #include "extern_example.h"
        
          function printGlobalVariableInOtherFile() {
             printf("value:%d", global_value); // "value=4"
          }
        

<div class="footnotes">
  <hr />
  
  <ol>
    <li id="fn-1">
      <p>
        <a href="http://shinluckyarchive.tistory.com/159">http://shinluckyarchive.tistory.com/159</a><a href="#fnref-1" rev="footnote">&#8617;</a>
      </p>
    </li>
  </ol>
</div>