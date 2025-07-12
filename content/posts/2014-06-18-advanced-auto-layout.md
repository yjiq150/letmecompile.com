---
title: iOS 고급 오토레이아웃(auto layout)
author: yjiq150
type: post
date: 2014-06-18T09:39:09+00:00
url: /advanced-auto-layout/
dsq_thread_id:
  - 2774600146
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - 팁
  - wwdc

---
본 글에서는 기본적인 오토레이아웃 튜토리얼들에서 잘 다루지 않는 커스텀 뷰에 오토레이아웃(auto layout)을 적용하는 방법과, 전반적으로 레이아웃 시스템이 어떻게 동작하는지를 중심으로 설명해보도록 하겠다. 오토레이아웃의 종류나 기본적인 적용방법들은 다른 [좋은 튜토리얼][1]들이 많으니 다루지 않을 예정이다.

## AutoLayout Programmatically 사용하기

개인적으로 오토레이아웃을 사용할 때도 인터페이스빌더를 되도록이면 쓰지 않으면서 코드만로 작성하는 방법을 선호한다. 애플에서 제공하는 `NSLayoutConstraint` 의 경우 가독성이 매우 떨어지는 단점이 있고, 이를 보완하기 위한 [Visual format language][2] 가 있지만, 이 또한 아주 직관적인 편은 아니며 문자열을 그대로 사용하기때문에 오타의 위험성도 존재한다. 이래저래 알아본 결과 [Masonry][3]라는 라이브러리가 그나마 가장 직관적이면서 가독성이 좋아 오토레이아웃 제약조건 코드를 작성할때 만족하며 사용중이다. Swift에서 사용을 원한다면 동일한 사람이 개발한 [SnapKit][4]을 사용하면 된다.

## 오토 레이아웃 진행 과정

  1. measurement pass(update constraints) &#8211; 레이아웃 변화나 사용자 입력 등 에의해 제약조건이 변경되어야 할 경우 이를 업데이트
  2. layout pass &#8211; 업데이트되어 설정된 제약조건에따라 뷰트리를 레이아웃
  3. display pass &#8211; 변화된 레이아웃에따라 다시 그려야 할 부분을 찾아 해당 부분을 다시 그림(draw: 관련 코드)

기본적으로 1, 2, 3 번호순으로 진행되지만, 레이아웃이 진행중에도 제약조건을 다시 업데이트하는 등 필요에따라 iterative하게 진행된다. 제약조건만으로는 표현이 불가능한 커스텀 레이아웃을 사용하는 경우가 이러한 iterative하게 진행되는 대표적인 예시이다.

## 제약조건만으로 표현이 불가능한 커스텀 레이아웃일 경우

상황: 슈퍼뷰의 사이즈가 줄어들때 뷰 안에있는 줄어든 사이즈에 맞춰 서브뷰중 하나를 삭제하고 싶다면?

### 구현 방법:

  1. 커스텀 레이아웃을 적용하고싶은 뷰에대해 `layout` (OS X) 또는 `layoutSubviews` (iOS) 오버라이드
  2. `[super layout]` 호출 (현재 적용되어있는 제약조건에 따라 해당 뷰의 레이아웃이 계산됨)
  3. 2번에 의해 새롭게 확정된 레이아웃을 조사하여 뷰가 원하는 대로 배치되어있는지 확인
  4. 원하는 대로 되어있지 않을경우 뷰 추가/삭제 및 제약조건을 업데이트한 후 `[super layout]` 다시 호출
  5. 원하는 형태로 모두 레이아웃될때까지 3~4번 과정 반복

### 코드 예시:

     - (void)layout {
          [super layout];
    
          // 슈퍼뷰의 바운드 벗어난 서브뷰삭제
          for( view in subviews ) {
               if(view is out of bound) {
                    [view removeFromSuperview];
                    [self updateConstraintsIfNeeded];
    
                    [super layout];
               }
          }
    
     }
    

### 주의할점

`layout` 또는 `layoutSubviews`는 오버라이드해서만 사용해야하고 직접 호출해서는 안된다.

만약 레이아웃 업데이트를 강제로 수행하고 싶다면 다음 두가지 방법중 하나를 필요에따라 선택하면 된다.

  * 즉시 업데이트가 필요한경우 `layoutSubtreeIfNeeded` 또는 `layoutIfNeeded`를 호출(여러곳에서 매번 즉시 업데이트를 수행하면 성능상에 문제가 생길 수 있음)
  * 다음 런루프에서 업데이트해도 괜찮은경우 setNeedsLayout 를 호출(여러곳에서 호출하더라도 다음런루프에서 최종적으로 한번만 레이아웃 업데이트를 수행)

## 오버라이드 포인트들이 호출되는 시점

위에서 설명했듯이 레이아웃 변경/제약조건 업데이트는 순차적으로만 일어나는것이 아니고 서로 상호적으로 필요에따라 번갈아가면서 호출되며 시스템 내부적으로 다뤄지는 부분이 많아 정확한 호출 시점을 잡아내는것이 쉽지 않다.

### `layoutSubview` / `layout` 이 언제 불리나?

  * (explicit) `setNeedsLayout` 호출된 경우 다음 런루프에 불림
  * (explicit) `layoutIfNeeded` 를 호출했을 때 제약조건 변화 등에의해 기존 레이아웃에서 변경될 점이 있다면 불림, 변경이 없는경우 불리지 않음.
  * (implicit) 뷰에 적용된 제약조건이 변경 될경우 특별히 `setNeedsLayout`을 명시적으로 호출하지 않아도 다음 런루프에서 자동으로 불림.

### `updateConstraints` 가 언제 불리나?

  * (explicit) `setNeedsUpdateConstraints` 호출된 경우 다음 런루프에 불림
  * (implicit) `layout` 진행 도중에 업데이트될 제약조건이 있는 경우 시스템 내부적으로 호출됨

## 오토레이아웃 코드 사용 패턴 예제

사용 패턴 1)

제약조건 관련 코드를 한곳에 모을 경우 (애플이 추천한 방식)

  * 커스텀 뷰의 제약조건 관련 코드를 오버라이드된 `updateConstraints` 메서드 안에 넣는다.
  * (iOS only) `UIViewController`의 경우 `updateViewConstraints` 를 오버라이드 하여 self.view의 제약조건을 관련 코드를 넣는것도 가능하다.
  * 뷰 구성이 변경되어 제약조건을 업데이트할 필요가 있을경우 업데이트가 필요한 뷰에 `setNeedsUpdateConstraints`을 호출하면 다음 런루프에 `updateConstraints` 메서드가 자동으로 호출되면서 제약조건을 업데이트 한다.
  * `setNeedsUpdateConstraints` 대신 `updateConstraintsIfNeeded`를 호출하면 바로 `updateConstraints`가 호출된다.(

사용 패턴 2)

애니메이션 사용시에는 그 즉시 애니메이션 완료시점의 레이아웃이 필요하기때문에 layoutIfNeeded를 애니메이션 블럭안에서 사용(다음 섹션 애니메이션 적용하기 참조)

## 애니메이션 적용하기

이미 설정된 제약조건중 `constant` 값만 바로 변경가능하기때문에 해당 제약조건 객체를 불러와서 `constant`값을 변경하면된다. 하지만 제약조건의 다른 속성값들(`multiplier`, `attribute` 등)은 readonly 속성이라 기존 설정된 제약조건 제거후 새로운 제약조건을 추가하는 방법으로 적용해야 한다.

### 제약조건 애니메이션(OS X only) 이용

  * 제약조건 자체에 animator 프록시 객체(OS X only)를 이용하여 constant값을 변경하는 애니메이션만 걸 수 있음
  * 애니메이션 도중에도 계속 제약조건을 만족시키면서 이동
  * 코어애니메이션보다는 조금 느리지만 충분히 빠름

### 코어애니메이션 이용

  * 제약조건 변경 후 layoutIfNeeded로 레이아웃을 변경하면 처음/끝 상태 기준으로 인터폴레이트(interpolate)되어 애니메이션 발생
  * 처음/끝 상태 기준만으로 애니메이션이 생성되므로 애니메이션 도중에 제약조건을 만족시키지 않는 레이아웃이 나타날 수 있음.
  * 속도 매우 빠름. 
      * (void) animateExample {  
        // 새 애니메이션을 진행하기 전에 완료되지 않은(pending) 레이아웃 작업들을 완료시키는 것을 추천  
        [self.view layoutIfNeeded];</p> 
        // 여기서 제약조건 변경  
        &#8230;
        
        // 애니메이션 적용  
        [UIView animateWithDuration:0.3 animations:^{  
        // 변경된 제약조건으로 바로 레이아웃을 진행  
        [self.view layoutIfNeeded];  
        }];
        
        }

## 커스텀 뷰(custom view) 만들기

애플에서 제공하는 기본 컨트롤들 외에, 직접 만든 커스텀뷰 에도 오토레이아웃이 적용되도록 할 수 있다. 이를 위해서 커스텀 뷰에 대해 오토레이아웃 계산에 필요한 몇가지 Alignment Rects와 Intrinsic Content Size 같은 해당 뷰의 영역을 표현해주는 메트릭(metric)들을 잘 정의해 두어야한다.

### Alignment Rects

커스텀 뷰를 만들때 뷰 안에 뱃지 같은 부수적인 컨트롤들이 추가되더라도 이는 실제 뷰의 정렬과는 크게 관계 없다. 따라서 제약조건의 경우 frame기준이 아닌 alignmentRect기준으로 적용된다. 커스텀 뷰 클래스에서 다음 두 메서드를 오버라이드 하여 이를 정의할 수 있다.

  * `alignmentRectForFrame:`
  * `frameForAlignmentRect:`

주의: 두 메서드가 리턴하는 값이 서로 인버스(inverse)관계를 만족하도록 정의해야한다.

### Intrinsic Content Size

UILabel처럼 해당 뷰가 갖고있는 기본 속성들만 갖고 해당 뷰의 프레임이 결정될 수 있는경우 `intrinsicContentSize` 메서드를 이용하여 가져올 수 있다.  
sizeToFit과 비슷하지만 더 정확하다.

intrinsic content size가 잘 정의된 경우, width/height를 따로 설정하지 않아도 제약조건 계산과정에서 width/height가 있는것 처럼 동작한다. 이미 UILabel의 텍스트 내용에 따른 너비/높이, 혹은 프로그레스바의 높이 등은 이미 잘 정의되어 있다. 그렇다면 커스텀 뷰에대한 `intrinsicContentSize`을 정의하려면 어떻게 하면 될까?

다음 세가지 사항을 통해 오토레이아웃에도 문제없이 동작하는 커스텀 뷰를 만들어 낼 수 있다.

  * `instrinsicContentSize`를 오버라이드하여 커스텀 뷰에 맞는 값을 계산하여 리턴해줄 수 있다.
  * `instrinsicContentSize`에 영향을 미치는 뷰의 속성 또는 컨텐츠가 바뀔때 `[self invalidateIntrinsicContentSize]` 를 호출해야한다. (이 메서드에서 오토레이아웃이 사이즈 제약조건이 다시 업데이트됨)</p> 
  * compression resistance & content hugging 정의
    
    &#8211;`intrinsicContentSize`가 정의된 뷰에대해서 컨텐트 부분보다 뷰 사이즈를 더 키우는 것을 선호하는지, 컨텐트 크기보다 뷰 사이즈가 줄어들어도 되는지 등을 설정하는 방법이다.

## 다국어 지원(Localization)

제약조건의 속성 중 leading, trailing <-> left, right 차이는 무엇일까?

leading은 일반적으로 left를 의미하지만, 오른쪽에서 왼쪽으로 쓰는 언어로 설정되어있는 경우에는 leading이 right를 의미한다. 따라서 다국어를 지원하는 어플리케이션을 개발할 경우 주의해서 사용해야 한다.

## 오토레이아웃 디버깅 팁

디버깅용 프라이빗 메서드인 _autolayoutTrace를 사용하면 ambiguous layout을 찾아내준다.

디버거에서 다음 메서드를 실행.

> po [[[UIApplication sharedApplication] keyWindow] _autolayoutTrace] 

     *<UIWindow:0x8d5e9b0> - AMBIGUOUS LAYOUT
     |   *<UILayoutContainerView:0xdc54f50>
     |   |   *<UITransitionView:0xdc64580>
     |   |   |   *<UIViewControllerWrapperView:0x8f909d0>
     |   |   |   |   *<UIView:0xdc7f450>
     |   |   |   |   |   *<_UILayoutGuide:0xdc54860> - AMBIGUOUS LAYOUT
     |   |   |   |   |   *<_UILayoutGuide:0xdc6c790> - AMBIGUOUS LAYOUT
     |   |   |   |   |   *<UIView:0xdc70390>
     |   |   |   |   |   |   *<UILabel:0xdc72410>
     |   |   |   |   |   |   *<UIImageView:0xdc72500>
     |   |   |   |   |   *<UIView:0x8f8eec0>
     |   |   |   |   |   |   *<UILabel:0x8f8ef50>
     |   |   |   |   |   |   *<UIImageView:0x8f8f380>
     |   |   |   |   |   *<UIButton:0x8f90310>
     |   |   |   |   |   |   <UIButtonLabel:0xdc78be0>
     |   |   <UITabBar:0x8d625e0>
     |   |   |   <_UITabBarBackgroundView:0x8ca0ea0>
     |   |   |   |   <_UIBackdropView:0x8c625e0>
     |   |   |   |   |   <_UIBackdropEffectView:0x8c9bd10>
     |   |   |   |   |   <UIView:0x8c981b0>
     |   |   |   <UITabBarButton:0x8d62ca0>
     |   |   |   |   <UITabBarButtonLabel:0x8d63540>
     |   |   |   <UITabBarButton:0x8d64700>
     |   |   |   |   <UITabBarButtonLabel:0x8d647e0>
     |   |   |   <UITabBarButton:0x8d64fe0>
     |   |   |   |   <UITabBarButtonLabel:0x8d650c0>
     |   |   |   <UIImageView:0x8cc8a00>
    

### 디버깅에 도움이 되는 NSUserDefaults 키값 설정

  * (OS X only) `NSConstraintBasedLayoutVisualizeMutuallyExclusiveConstraints` &#8211; 제약조건에 문제가 생길때 자동으로 [NSWindow visualizeConstraints:] 호출해서 비주얼라이즈 활성화.</p> 
  * `UIViewShowAlignmentRects/NSViewShowAlignmentRects` &#8211; alignmentRect를 비주얼라이즈

  * `NSForceRightToLeftWritingDirection` &#8211; 강제로 오른쪽에서 왼쪽으로 쓰는 언어처럼 시뮬레이트

  * `NSDoubleLocalizedStrings` &#8211; 스트링 내용을 강제로 두번 반복해서 단어길이가 긴 언어 테스트 할 수 있게해줌

## 오토레이아웃 관련 오픈소스 라이브러리

  * https://github.com/Masonry/Masonry (추천)

  * https://github.com/iMartinKiss/KeepLayout

  * https://github.com/ReactiveCocoa/ReactiveCocoaLayout

## 참고자료

http://www.objc.io/issue-3/advanced-auto-layout-toolbox.html  
https://developer.apple.com/library/ios/documentation/userexperience/conceptual/AutolayoutPG/AutoLayoutbyExample/AutoLayoutbyExample.html  
https://medium.com/@jsleeuw/mastering-programmatic-auto-layout-b02ed2499d79

 [1]: http://www.raywenderlich.com/ko/21139/ios-6%EC%97%90%EC%84%9C-%EC%98%A4%ED%86%A0-%EB%A0%88%EC%9D%B4%EC%95%84%EC%9B%83-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%ED%8C%8C%ED%8A%B8-1-1
 [2]: https://developer.apple.com/library/ios/documentation/userexperience/conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH3-SW1
 [3]: https://github.com/Masonry/Masonry
 [4]: https://github.com/SnapKit/SnapKit