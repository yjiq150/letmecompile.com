---
title: Xcode LLDB 디버깅 테크닉
date: 2014-02-28T09:19:37+00:00
url: /xcode-lldb-디버깅-테크닉/
dsq_thread_id:
  - 2334422928
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/scheduler-cron-tutorial/"     class="post-725"><span class="crp_title">작업 스케쥴러 크론(Cron) 간단 사용법</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/scheduler-cron-tutorial/"     class="post-725"><span class="crp_title">작업 스케쥴러 크론(Cron) 간단 사용법</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - cocoa
  - iOS

---
Xcode에 기본으로 내장되어있는 디버거(debugger)인 LLDB는 기존에 가장 많이 사용되던 gdb보다 많은 유용한 기능들을 갖고있다. LLDB 명령어 분석기에는 파이썬(Python) 인터프리터가 내장되어있어서 `script` 명령어로 파이썬 코드들을 사용할 수 있다. 이 또한 반대로 모든 LLDB API들이 SB(Scripting Bridge) 를 통해서 Python에서도 사용이 가능하도록 되어있다.[^1] 이러한 파이썬과의 연계를 통해서 Xcode 디버깅을 진행 할때 더 세분화된 조건들을 정의하여 브레이크포인트(breakpoint)에 지정할 수 있게되었고, 이런 조건에따라 액션 수행, 브레이크포인트 무시/진행여부 결정 등의 다양한 동작이 가능하다[^2].  
본 글은 애플의 &#8220;Advance Debugging with LLDB (WWDC 2013, Session 413)&#8221; 발표자료[^3]를 토대로 요약 및 정리한 내용이다.

## 기본 명령

LLDB에서 기본적인 GDB 스타일의 명령어를 대부분 사용가능하다. GDB에 어느정도 익숙하다면 동일 명령어를 입력하여 LLDB를 문제없이 사용가능하고, 잘 모르겠으면 [명령어 비교 표][1]를 참고하도록 하자.

### 값 확인하기(Evaluating expression)

가장 기본적으로 많이 사용하는 명령어는 변수값 확인할때 사용하는 `p`와 `po` 명령어이다.  
실제 LLDB에서의 커맨드는 `expr` 이지만 GDB에서 사용하던 명령어대로 `p`, `po`를 입력해도 잘동작한다.

C/C++에서 사용되는 원시타입(primitive type) 변수나 구조체 변수는 `p [변수명]` 명령어를 이용하여 다음과 같이 해당 변수값의 출력이 가능하다.

    (lldb) p rect
    

Objective-C 객체의 경우 `po [변수명]`을 이용하면 해당 객체에 `description` 메시지가 보내져서 해당 객체에 대한 적절한 정보가 출력되게된다.

    (lldb) po myString
    

### 흐름 제어하기

`c` 명령어를 통해 브레이크포인트를 떠나서 계속 진행(continue) 할 수 있다.

    (lldb) c
    

## 조건에따른 액션 지정

특정 브레이크포인트에 진입했을 때 멈추지 않고 특정 명령을 수행후 바로 진행하는 액션들을 지정하는 것이 가능하다.

    (lldb) br co a 
    (lldb) breakpoint command add
    Enter your debugger command(s).  Type 'DONE' to end.
    > p rect
    > bt
    > c
    > DONE
    

위와같이 입력하면 현재 멈춰있는 브레이크포인트에 다음번에 진입시 등록한 명령어가 자동으로 수행된다. (rect 출력, backtrace 출력 후 브레이크없이 넘어가기)

## 특정 인스턴스에 브레이크 포인트 걸기

하나의 클래스 정의로부터 여러개의 인스턴스를 생성하여 사용중인 상황에서, 여러 인스턴스들 중 특정 인스턴스에 대해서만 브레이크포인트를 걸어서 상태를 확인하고 싶을 경우가 있다. 이때는 다음과 같은 명령어들을 통해서 미리 해당 인스턴스의 레퍼런스를 저장해 둔 후 브레이크포인트에 조건을 지정하는 방식으로 접근하면 된다.

브레이크가 걸린 상태에서 특정 인스턴스의 레퍼런스를 `$myModel`이라는 LLVM 변수에 저장한다

    (lldb) p id $myModel = self
    

아래 명령어로 컨디셔널 브레이크포인트를 건다(Xcode GUI에서도 해당 브레이크 포인트에 오른쪽버튼을 클릭하여 편집하면 조건을 지정할 수 있으며, 조건식에 앞서 지정한 LLVM 변수 `$myModel` 또한 동일하게 사용이 가능하다.)

    (lldb) br m -c “self == $myModel”
    (lldb) breakpoint modify —condition “self == $myModel”
    

## 특정 변수 값 변화 모니터링하기

`watchpoint` 명령어를 이용하면 특정 변수값이 바뀔 때 break를 걸리게 할 수 있다. 이 명령어의 경우 모니터링 하는데 많은 CPU자원이 소모되기 때문에 성능상의 문제로 모니터링할 수 있는 변수의 갯수는 Intel 4개, ARM 2개로 제한되어 있다.

    (lldb) w s v self->myVariable
    (lldb) watchpoint set variable self->myVariable 
    

## 실행중에 값 바꾸기/함수 호출하기(Calling code by hand)

프로그램이 실행중에 특정 상황을 만들어내지 못해서 디버깅이 어려운 경우, 실행중에 메모리상의 값을 바꾸거나, 수동으로 직접 호출하는 것이 필요한 경우들이 많이 있다. 이를 위해 LLDB에서는 실행중에 브레이크가 걸린 상태에서 특정코드를 on-the-fly로 실행하거나 메모리상의 값을 변경하는것이 가능하다. 객체내부의 어딘가에서 브레이크가 걸려 있을 때 다음명령어를 이용하면 해당 코드를 실행하는것이 가능하다.

    (lldb) e -i false — [self thisIsCalledByHand]
    

17번라인에 브레이크가 걸려있는 상태에서 위 명령어를 실행시 아래와같은 결과를 볼 수 있다.

[<img loading="lazy" width="701" height="700" src="/uploads/2014/02/lldb_debug_example.png" alt="lldb_debug_example" class="alignnone size-full wp-image-290" />][2]

## 커스텀 객체에 대한 값 확인을 위한 형식 지정하기

Xcode에서 STL, Foundation, CoreFoundation 등의 객체는 사용자에게 읽을 수 있는 형태로 제공되고 있기때문에 디버깅 윈도우에서 손쉽게 값을 확인하는 것이 가능하다 하지만 사용자가 만들어낸 커스텀 객체는 raw 데이터 형태로만 조회되는 경우가 많기 때문에 디버거에서 유용한 정보를 바로 보는것이 쉽지 않다. 이런경우 LLDB에 내장되어있는 파이썬을 이용하여 데이터 형식지정자(Data Formatter)를 등록하여 편리하게 사용할 수 있다.

애플 WWDC 2013 LLDB세션 비디오에 나오는 다음 예시를 살펴보자. `MyAddress`라는 커스텀 클래스를 사용할경우 해당 변수에대한 요약정보(summary)가 나타나지 않아서 불편하다.

[<img loading="lazy" width="380" height="203" src="/uploads/2014/02/formatter_before.png" alt="formatter_before" class="alignnone size-full wp-image-289" />][3]

이때 이를 원하는 형식으로 나타나게 하려면 먼저 형식을 나타내기위한 파이썬 함수를 다음과같이 정의한다.

    def MyAddress_Summary(value,unused):
        firstName = value.GetChildMemberWithName("_firstName")
        lastName = value.GetChildMemberWithName("_lastName")        firstNameSummary = firstName.GetSummary()
        lastNameSummary = lastName.GetSummary()
        # process the data as you wish
        return firstNameSummary + “ “ + lastNameSummary
    

다음 명령어를 실행해서 방금 전에 파이썬 스크립트로 정의했던 형식지정자 함수를 등록해준다.

    (lldb) ty su a MyAddress -F MyAddress_Summary
    (lldb) type summary add MyAddress --python-function MyAddress_Summary
    

이 후에 실행하면 다음과같이 해당 변수의 요약정보가 원하는대로 변화되어 있는 것을 확인 할 수 있다.  
[<img loading="lazy" width="381" height="204" src="/uploads/2014/02/formatter_after.png" alt="formatter_after"  class="alignnone size-full wp-image-288" />][4]

LLDB가 등록된 `MyAddress_Summary(value,unused)` 파이썬 함수를 호출할때 value 파라메터는 실제 `SBValue`라는 객체로되어있다. 이 `SBValue`는 정의되어있는 LLDB 객체모델들 중 하나로써, LLDB가 갖고있는 현재 프로그램 디버깅 상태정보들을 파이썬 스크립트와 교환하는데 이용되는 객체이다.

<div class="footnotes">
  <hr />
  
  <ol>
    <li id="fn:1">
      <p>
        http://lldb.llvm.org/python-reference.html<a href="#fnref:1" rev="footnote">&#8617;</a>
      </p>
    </li>
    <li id="fn:2">
      <p>
        https://developer.apple.com/librarY/mac/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/Introduction.html<a href="#fnref:2" rev="footnote">&#8617;</a>
      </p>
    </li>
    <li id="fn:3">
      <p>
        https://developer.apple.com/wwdc/videos/<a href="#fnref:3" rev="footnote">&#8617;</a>
      </p>
    </li>
  </ol>
</div>

 [1]: http://lldb.llvm.org/lldb-gdb.html
 [2]: /uploads/2014/02/lldb_debug_example.png
 [3]: /uploads/2014/02/formatter_before.png
 [4]: /uploads/2014/02/formatter_after.png
