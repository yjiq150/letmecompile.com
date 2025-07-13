---
title: Swift 익스텐션, Obj-C 카테고리 메서드명 Prefix하기
date: 2017-09-23T14:51:47+00:00
url: /swift-extension-obj-c-category-method-prefix/
dsq_thread_id:
  - 6165086065
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - 카테고리
  - 익스텐션
  - 이름 충돌

---
## Obj-C category의 위험성

Obj-C에서 동일한 메서드 이름을 가진 카테고리(category) A, B가 동시에 존재하더라도, 컴파일 타임에 오류가 발생하지 않고 정상적으로 빌드되어 실행이 가능하다. 하지만 런타임에서 A와 B중 어떤 녀석이 먼저 호출될지 알 수 없기때문에 (규칙이 없으며 랜덤하게 호출됨) 매우 위험한 상황이 발생한다. 이를 방지하기 위해서 Obj-C에서 카테고리 메서드에는 프로젝트에서 사용하는 고유의 prefix를 붙여줘서 이름이 완전히 겹칠 확률을 최대한 줄여주는 방식을 꼭 사용해야 한다. 그렇지 않으면 외부 라이브러리를 사용할때 해당 라이브러리에 동일한 category명이 존재해서 랜덤하게 호출되는 상황이 발생 할 수 있다. 이는 매우 원인 파악도 힘든 에러가 될 것이기에 꼭 피해야 한다.

  * 문제상황) `UIImage+MyProject.h`와 `UIImage+ExternalProject.h`가 동시에 import되어있는 상황에서 `[UIImage from:]`을 호출하는 경우 누가 불릴지 아무도 장담할 수 없다. 
        // UIImage+MyProject.h
        @interface UIImage (MyProject)
        + (nonnull UIImage*)from:(nonnull UIColor*)color;
        @end
        
        // UIImage+ExternalProject.h
        @interface UIImage (ExternalProject)
        + (nonnull UIImage*)from:(nonnull UIColor*)color;
        @end
        

  * 해결) category method에 prefix를 붙여준다 
        // UIImage+MyProject.h
        @interface UIImage (MyProject)
        + (nonnull UIImage*)myp_from:(nonnull UIColor*)color;
        @end
        
        // UIImage+ExternalProject.h
        @interface UIImage (ExternalProject)
        + (nonnull UIImage*)from:(nonnull UIColor*)color;
        @end
        

## Swift extension은 어떨까?

그렇다면 Swift extension을 정의할때도 Obj-C에서 처럼 메서드 이름을 지을때 주의를 해야할지 알아보도록 하자.

Swift의 경우 Obj-C와 달리 module개념이 생겼기때문에 같은 Swift module내에 존재하는 extension에 동일한 메서드 이름 가진 메서드가 존재할 경우 바로 컴파일 에러가 발생한다.

하지만 여전히 Obj-C 코드와 섞이는 경우 무엇인가 불안하다. 아래와같이 완전히 동일한 메서드 명을가진 swift extension과 obj-c category를 각각 같은 프로젝트에서 컴파일을 진행할때는 어떻게 될까? 혹시나 문제가 없을지 궁금해서 일부러 동일한 메서드명을 갖도록 만들어서 실험을 해 보았다. (앞에서 언급했듯이 실제로는 Obj-C 카테고리를 메서드를 정의할때는 꼭 prefix없이 만드는 것이 좋다.)

  * Swift version 
        extension UIImage {
            static func from(_ color: UIColor) -> UIImage {
                return UIImage()
            }
        }
        

  * Obj-C version 
        @interface UIImage (MyProject)
        + (nonnull UIImage*)from:(nonnull UIColor*)color;
        @end
        
        @implementation UIImage (MyProject)
        + (nonnull UIImage*)from:(nonnull UIColor*)color {
            return [UIImage new];
        }
        @end
        

### 두 함수가 모두 정의된 채로 컴파일 되었을때

메서드 이름이 완전히 동일함에도 불구하고 컴파일 에러는 나지 않았다. Obj-C의 category와 이름이 겹치는 것은 컴파일러에서 잡아주지 못하는듯 하다.

런타임에서는 아래와같이 약간 신기하게 동작한다.  
1. Swift에서 `UIImage.from(UIColor.clear)`를 호출시 swift version의 함수가 호출됨  
2. Obj-C에서 `[UIImage from:[UIColor clearColor]]` 호출시 Obj-C version의 함수가 호출됨

### 둘 중 어느 한쪽만 정의된 채로 컴파일 되었을때

여전히 문제 없이 잘 컴파일 되며 Swift에서 호출하던, Obj-C에서 호출하던 유일하게 정의된 해당 메서드가 문제없이 호출된다. (Obj-C category/Swift extension 둘다 동일하게 동작)

## 결론

Swift extension을 작성할때는 prefix를 붙일 걱정을 할 필요가 없다. Swift간에는 컴파일러가 알아서 에러를 뱉어줄 것이고, Obj-C의 category에만 잘 prefix가 붙어있다면 Swift와 Obj-C사이에 이름이 겹칠 가능성은 없기 때문이다.