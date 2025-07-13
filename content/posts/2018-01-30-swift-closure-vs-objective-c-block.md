---
title: Swift Closure vs. Objective-C Block 차이점 비교 분석
date: 2018-01-30T03:07:08+00:00
url: /swift-closure-vs-objective-c-block/
dsq_thread_id:
  - 6446809267
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
Obj-C의 블락(block)이나 Swift 클로저(closure)는 컨셉은 거의 동일하나 closure 내부에서 현재 scope에 존재하는 값 타입(value type) 변수들을 캡쳐(capture)해서 사용할 때 **기본동작이 반대**로 되어있기 때문에 사용법에 주의를 기울여야 한다. 반면 클래스 인스턴스와 같은 참조 타입(reference type) 변수들은 항상 reference copy가 일어나기 때문에 두 언어를 사용할때 차이점에 크게 신경쓰지않아도 된다. (대신 retain cycle이 생기지 않도록 조심해야 한다.)

다음 예제들을 통해서 차이점을 좀 더 자세히 알아보자.

## Capture in Swift closure

Swift의 경우 일반적으로 value type 변수를 다른 변수에 할당하거나, 함수 파라메터로 전달 할 때 copy동작이 기본이다. 하지만 closure에서 변수를 capture를 할 때는 명시적인 capture list를 작성하지 않으면 **value type 변수(struct 변수 포함) 임에도 불구하고 reference capture**가 일어난다. 확인을 위해서 다음 예제를 살펴보자.

    var anInteger = 42
    
    let testClosure = {
        // anInteger는 capture되는 순간 reference copy됨
        print("Integer is: \(anInteger)")
    }
    
    anInteger = 84
    
    testClosure()
    // Prints "Integer is 84"
    

경우에 따라서 referece copy동작 대신 value copy가 필요할 때도 있다. 이 경우 아래와 같이 `[anInteger, ...]` 형태로 capture list를 만들어서 변수를 명시 해주면 된다. (value capture된 값은 const value type 형태로 capture가 되고 closure안에서 변경이 불가능해진다.)

    var anInteger = 42
    
    let testClosure = { [anInteger] in
        // anInteger는 capture되는 순간 value copy됨
        print("Integer is: \(anInteger)")
    }
    
    anInteger = 84
    
    testClosure()
    // Prints "Integer is 42"
    

## Capture in Objective-C Block

반대로 Obj-C block의 경우 value type이 capture 될때 기본동작이 value copy 이다. 아래 예제를 살펴보자.

    int anInteger = 42;
    
    void (^testBlock)(void) = ^{
         // anInteger는 capture되는 순간 value copy됨
        NSLog(@"Integer is: %i", anInteger);
    };
    
    anInteger = 84;
    
    testBlock();
    // Prints "Integer is: 42"
    

이를 reference copy로 변경 하려면 __block 키워드를 capture할 변수 선언시에 명시해주면 된다.

    __block int anInteger = 42;
    
    void (^testBlock)(void) = ^{
         // anInteger는 caputure되는 순간 reference copy됨
        NSLog(@"Integer is: %i", anInteger);
    };
    
    anInteger = 84;
    
    testBlock();
    // Prints "Integer is: 84"
    

Obj-c의 블록 사용법에 관해서 더 자세한 내용은 다음 글에 더 자세히 설명해두었다. [Objective-C Block 동작 심층 분석][1]

 [1]: https://www.letmecompile.com/objective-c-block-%EB%8F%99%EC%9E%91-%EC%8B%AC%EC%B8%B5-%EB%B6%84%EC%84%9D/