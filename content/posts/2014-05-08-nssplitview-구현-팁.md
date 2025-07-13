---
title: NSSplitView 구현 팁
date: 2014-05-08T13:21:15+00:00
url: /nssplitview-구현-팁/
dsq_thread_id:
  - 2669924627
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa

---
맥용 코코아(Cocoa) 어플리케이션을 만들다보면 `NSSplitView`가 많이 사용되는데 이와 관련된 몇가지 팁들을 소개해본다.

## NSSplitView 디바이더(divider) 숨기기

NSSplitView를 서브클래싱 한 후 다음 메서드를 오버라이드하여 디바이더의 두께를 0으로 만들어 숨길 수 있다.

    -(CGFloat)dividerThickness {
       return 0;
    }
    

## 디바이더를 숨겨도 리사이즈가 가능하게 만들기

위와같이 디바이더의 두께를 0으로 지정한경우, 디바이더의 영역이 없기때문에 스플릿뷰를 리사이즈하는 동작이 불가능하게된다. 이때 디바이더 대신 특정영역을 지정하여 해당 영역을 사용자가 마우스로 클릭을 할경우 리사이즈되도록 할수있다.

아래의 delegate를 구현한 후 divider index에 맞춰서 클릭가능 영역을 지정하여 리턴해주면 된다.

    - (NSRect)splitView:(NSSplitView *)aSplitView additionalEffectiveRectOfDividerAtIndex:(NSInteger)dividerIndex
    {
        // return rect that will receive resize command by mouse
        return NSRectMake(0, 0, 100, 100);
    }
    

## 사용자 액션이 아닌 코드로 리사이즈하기

사용자가 디바이더를 드래그해서 사이즈를 조정할 경우 자동으로 디바이더에 인접한 뷰가 resize되게 된다. 이를 프로그램 코드로(programmatically) 조정하고 싶은 경우에는 어떻게 하면될까?

다음 코드와같이 splitView의 각각의 뷰를 `setFrame:`을 통해 원하는 사이즈로 조정한 후, 마지막으로 splitView에대해 adjustSubviews 메서드를 불러주면 해당 서브뷰의 사이즈에 맞춰 디바이더 위치가 조정된다. 이때 애니메이션을 주고싶다면 `setFrame:`을 `NSAnimationContext` 범위안에두고 할때 `animator`를 사용하여 `setFrame:`을 해주면 된다.

    - (void)setColumn1:(NSRect)rectForCol1 column2:(NSRect)rectForCol2 {
        [[self.subviews objectAtIndex:0] setFrame:rectForCol1];
        [[self.subviews objectAtIndex:1] setFrame:rectForCol2];     
        [self adjustSubviews];
    }
    

## 스플릿뷰가 리사이징 될때 서브뷰들이 비율 유지

스플릿뷰가 리사이징될때 특정 서브뷰에 최소값을 지정하고싶다거나, 서브뷰들이 비율을 유지하면서 리사이징이 되게 하려면 다음 글을 참조하자. [우선순위-기반-리사이징-예제][1]

 [1]: http://www.letmecompile.com/nssplitview-우선순위-기반-리사이징-예제