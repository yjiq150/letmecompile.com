---
title: Objective-C Tips from WWDC 2013
date: 2014-05-19T14:38:03+00:00
url: /objective-c-tips-from-wwdc-2013/
dsq_thread_id:
  - 2696962772
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/swift-struct-vs-class-%ec%b0%a8%ec%9d%b4%ec%a0%90-%eb%b9%84%ea%b5%90-%eb%b6%84%ec%84%9d/"     class="post-706"><span class="crp_title">Swift struct vs. class 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/swift-closure-vs-objective-c-block/"     class="post-704"><span class="crp_title">Swift Closure vs. Objective-C Block 차이점 비교 분석</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
애플 WWDC2013, Session 228에 소개된 내용들 중 iOS나 Mac 개발시 자주쓰일만한 팁들을 추려내서 간단히 정리해보았다.

## 서브스크립팅(subscripting) 사용하기

Modern Objective-C 문법이 적용되면서 NSDictionary나 NSArray같은 foundation 데이터 구조객체들은 서브스크립팅을 이용하여 다음과같이 간결하게 작성이 가능하게되었다.

`[myArray objectAtIndex:3]` -> `myArray[3]`  
`[myDictionary objectForKey:@"key"]` -> `myDictionary[@"key"]`

사용자가 만든 커스텀 클래스에도 서브스크립팅을 위한 set/get메서드만 잘 정의해두면 훨씬 더 간결한 프로그램 작성이 가능하다.

MyCustomArray 클래스에

     - (id)objectAtIndexedSubscript:(NSUInteger)idx;
     - (void)setObject:(id)obj atIndexedSubscript:(NSUInteger)idx;
    

위 두가지 메서드 구현시 다음과같이 배열처럼 인덱스로 바로 접근 가능

get: `NSLog(@“%@“, myCustomInstance[1]);`

set: `myCustomInstance[0] = @“test";`

딕셔너리(dictionary)의 경우 아래 두가지 메서드를 구현해주면 된다.

     - (id)objectAtKeyedSubscript:(id <NSCopying>)key;
     - (void)setObject:(id)obj forKeyedSubscript:(id <NSCopying>)key;
    

## NSExpression: 문자열로된 수식 계산

스트링(string)형태로 된 수식을 계산하고 싶을때 유용하게 사용될 수 있다.

     NSString *text = @"3 + 5 * 4e10";
     NSExpression *e = [NSExpression expressionWithFormat:text, nil];
     NSNumber *result = [e expressionValueWithObject:nil context:nil];
     NSLog(@"result: %@", result);
    

위 코드의 출력값은 `result: 200000000003` 이다.

## NSArray reverse ordering

`reverseObjectEnumerator`를 사용하면 손쉽게 reverse array를 만들 수 있다.

    NSArray *numbers = @[ @1, @2, @3 ];
    NSArray *reversed = numbers.reverseObjectEnumerator.allObjects;
    

## NSNumber vs. NSValue

Objective-C를 접한지 얼마 안된경우 이 비슷한 이름을 가진 두가지 클래스의 차이점이 뭔지 헷갈릴경우가 많다.

구분하자면,

  * NSNumber의경우 int, float, bool 등의 원시 타입(primitive type)을 객체화시켜 저장하는데 사용
  * NSValue의 경우 scalar값들이나 struct (NSRange, CGPoint, NSRect 등)을 객체화시켜 저장

예1)

    NSMutableArray *array = [@[] mutableCopy];
    array[0] = [NSValue valueWithPoint:CGPointZero];
    array[1] = [NSValue valueWithRange:NSMakeRange(3, 17)];
    

예2) 사용자정의 구조체

    typedef struct RGB {
         float red, green, blue;
    } _RGB;
    
    RGB color = {1.0f, 0.0f, 0.0f};
    array[2] = [NSValue valueWithBytes:&color objCType:@encode(RGB)];
    

## NSString이 아닌 객체를 dictionary의 key로 사용하기

딕셔너리에 키(key)에 대해 값(value)를 셋팅할경우, 키는 항상 `copy`되어 사용되기때문에 키로 사용되는 클래스는 항상 `NSCopying` 프로토콜을 따라야한다. 일반적으로 딕셔너리의 키값으로 많이 사용되는 `NSString`의 경우 이 프로토콜을 따르고있다.

`myObject` 객체가 `NSCopying` 프로토콜을 따를경우 에는 다음과 같이 사용해도 아무 문제가 없지만

    mutableDic[myObject] = @42;
    

`NSCopying` 프로토콜을 따르지 않을경우 크래쉬가 발생한다. 이때 해당 객체가 `NSCopying`프로토콜을 따르도록 수정해도되지만, 해당객체가 다른 라이브러리에서와서 직접적인 수정이 어려운 경우 다음과같이 `NSValue` 객체로 한번 wrapping 해주면 문제가 해결된다(`NSValue`가 `NSCopying`프로토콜을 따르기 때문).

    keyObject = [NSValue valueWithNonRetainedObject:myObject]
    mutableDic[keyObject] = @42;
    

`valueWithNonRetainedObject:` 메서드의 경우 해당 객체(object)를 retain하지 않는다는 점에 주의해야 한다.

## NSDataDetector

자주사용되는 특정 스트링 패턴을 긴 문자열내에서 손쉽게 추출해주는 라이브러리.

  * Dates
  * Addresses
  * Links
  * Phone numbers 
    NSString *string = @&#8221;123 Main St. / (555) 555-5555&#8243;;  
    NSError *error;  
    NSDataDetector *detector = [NSDataDetector  
    dataDetectorWithTypes:NSTextCheckingTypeLink |  
    NSTextCheckingTypePhoneNumber  
    error:&error];  
    [detector enumerateMatchesInString:string  
    options:kNilOptions  
    range:NSMakeRange(0, [string length])  
    usingBlock:^(NSTextCheckingResult *result, NSMatchingFlags flags,  
    BOOL *stop) {  
    NSLog(@&#8221;Match: %@&#8221;, result);  
    }];