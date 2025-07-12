---
title: NSSplitView 우선순위 기반 리사이징 예제
author: yjiq150
type: post
date: 2013-08-05T15:57:40+00:00
url: /nssplitview-우선순위-기반-리사이징-예제/
dsq_thread_id:
  - 1572658783
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - CocoaWithLove 번역
tags:
  - NSSplitView
  - NSSplitView multiple column
  - NSSplitView 다중 컬럼
  - 리사이징
  - NSSplitView 예제
  - NSSplitView 튜토리얼
  - 에버노트 NSSplitView 구현
  - NSSplitView 리사이징
  - CocoaWithLove

---
`NSSplitView`의 경우 각 컬럼이 비율을 유지하면서 리사이징 되는것이 기본값이다. 즉 `NSSplitView`의 크기가 변할때 각 컬럼이 동일 비율로 증가하게되는데, 이러한 방식은 사이드바를 가지는 UI(예: 아이튠즈나 엑스코드의 사이드 바)에는 적합하지 않다. 이 포스트에서는 우선순위 리스트에 기반한 방법으로 사이드바와 메인 뷰를 적절히 리사이징 하는 delegate 클래스를 다뤄보도록 하겠다.

## 비율 vs. 우선순위 리사이징

3개의 뷰를 가지는 NSSplitView는 다음과 같이 동작한다:

비율 리사이징의 경우, 윈도우 크기가 늘어나면 각 컬럼이 너비가 동일 비율로 증가한다.

[<img loading="lazy" width="370" height="124" src="/uploads/2013/08/proportionalsmall.png" alt="proportionalsmall" class="alignnone size-medium wp-image-121" />][1]  
[<img loading="lazy" width="563" height="124" src="/uploads/2013/08/proportionallarge.png" alt="proportionallarge" class="alignnone size-full wp-image-123" />][2]

우선순위 리사이징의 경우, 다른 뷰들은 그대로 있고 우선순위가 가장 높은 뷰의 크기가 먼저 늘어나게 된다.

[<img loading="lazy" width="427" height="124" src="/uploads/2013/08/prioritysmall.png" alt="prioritysmall" class="alignnone size-full wp-image-120" />][3]  
[<img loading="lazy" width="550" height="124" src="/uploads/2013/08/prioritylarge.png" alt="prioritylarge" class="alignnone size-full wp-image-122" />][4]

> 우선순위 리사이징에 대한 샘플프로젝트는 다음 링크에서 다운로드 가능하다. [ColumnSplitView.zip (60kb)][5]

## 스플릿뷰 크기를 줄이는 경우의 동작

우선순위 리사이징의 경우 늘어날때도 우선순위가 높은 메인 뷰가 먼저 늘어나지만, 줄어들때도 메인 뷰가 먼저 줄어든다. 따라서 메인뷰의 사이즈가 0이되지 않도록 최소 크기또한 정해놓을 필요가 있다.

가장 높은 우선순위의 메인 뷰가 최소 크기에 도달하면, 해당 뷰의 크기는 더이상 줄어들지 않고 남은 다른 뷰들의 크기가 줄어들게 된다. 남은 뷰들에 대해서도 동일한 알고리즘을 통해 우선순위와 최소크기에 따라 리사이징이 된다. 또한 NSSplitView를 포함하고 있는 window나 상위 뷰는 모든 컬럼들이 최소 크기인 경우의 총 크기보다 작아지지 않도록 제한되어야 한다.

## `NSSplitView` 다루기

`NSSplitView`의 delegate 메서드들을 이용하여 각 섹션별 최소 크기의 설정과 어떤 뷰가 리사이즈되어야 할 지 결정할 수 있다.

예제파일에 포함된 `PrioritySplitViewDelegate` 클래스를 NSSplitView에 delegate로 등록하게되면 앞서 설명한 우선순위 기반의 리사이징이 가능하게 된다.

<pre>@interface PrioritySplitViewDelegate : NSObject
{
    NSMutableDictionary *lengthsByViewIndex;
    NSMutableDictionary *viewIndicesByPriority;
}

- (void)setMinimumLength:(CGFloat)minLength
    forViewAtIndex:(NSInteger)viewIndex;
- (void)setPriority:(NSInteger)priorityIndex
    forViewAtIndex:(NSInteger)viewIndex;

@end
</pre>

delegate가 스플릿뷰에 얼마나 많은 서브뷰들이 있는지 미리 모르기 때문에, 사용 전에 미리 뷰와 우선순위를 지정해야 한다. 모든 뷰에 우선순위를 지정하지 않으면 `NSSplitView`가 리사이징될 때 예외가 발생할 것이다. 최소크기의 경우 optional 이고, 직접 설정하지 않으면 기본값은 0이 된다.

## 구현

`NSSplitView`의 경우 **divider를 드래그하여 조정할때**와 **스플릿뷰가 리사이징 될 때** 에 대한 delegate 메서드들을 제공한다.

사용자가 스플릿뷰의 **divider를 드래그하여 조정할때** delegate 메서드인`splitView:constrainMinCoordinate:ofSubviewAt:` 와 `splitView:constrainMaxCoordinate:ofSubviewAt:`가 호출된다. 이 메서드 안에서 우리는 뷰의 최소 크기를 제어할 수 있다.

두개의 컬럼 사이를 나누고 있는 divider를 왼쪽으로 드래그할경우, 왼쪽에있는 뷰가 줄어들게 되므로 여기에 대한 최소크기를 리턴해주어야 한다.

<pre>- (CGFloat)splitView:(NSSplitView *)sender
    constrainMinCoordinate:(CGFloat)proposedMin ofSubviewAt:(NSInteger)offset
{
    NSView *subview = [[sender subviews] objectAtIndex:offset];
    NSRect subviewFrame = subview.frame;
    CGFloat frameOrigin;
    if ([sender isVertical])
    {
        frameOrigin = subviewFrame.origin.x;
    }
    else
    {
        frameOrigin = subviewFrame.origin.y;
    }

    CGFloat minimumSize =
        [[lengthsByViewIndex objectForKey:[NSNumber numberWithInteger:offset]]
            doubleValue];

    return frameOrigin + minimumSize;
}
</pre>

`splitView:constrainMaxCoordinate:ofSubviewAt:` 의 경우도 비슷하다. 자세한 내용은 예제 프로젝트에서 확인 할 수 있다.

마지막으로 우선순위를 지정해야하는데, 이 부분은 `splitView:resizeSubviewsWithOldSize` 에서 설정 가능하다. 이 delegate 메서드는 **스플릿뷰가 리사이징** 될때 호출된다.(일반적으로 스플릿뷰를 포함한 상위 부모뷰나 윈도우가 리사이징 될 경우)

알고리즘은 다음과 같다:

  1. 우선순위로 정렬된 서브뷰들의 리스트에 대해 이터레이션 한다
  2. 전체 크기변화에 의한 변화를 각 뷰에 대해서 적용한다.
  3. 각 뷰에 새로운 크기를 적용했을때 미리 셋팅된 최소크기보다 작아질경우, 해당 뷰에 대해 크기변화를 적용할수 있는 만큼 최대한 적용한 후, 우선순위에 의해 다음뷰로 넘어가서 나머지 크기변화를 적용한다. 다음 뷰로 넘어가는 스플릿뷰의 크기 변화는 코드에서 `delta`로 표현된다.

<pre>for (NSNumber *priorityIndex in [[viewIndicesByPriority allKeys] sortedArrayUsingSelector:@selector(compare:)])
{
    NSNumber *viewIndex = [viewIndicesByPriority objectForKey:priorityIndex];
    NSInteger viewIndexValue = [viewIndex integerValue];
    if (viewIndexValue &gt;= subviewsCount)
    {
        continue;
    }

    NSView *view = [subviews objectAtIndex:viewIndexValue];

    NSSize frameSize = [view frame].size;
    NSNumber *minLength = [lengthsByViewIndex objectForKey:viewIndex];
    CGFloat minLengthValue = [minLength doubleValue];

    if (isVertical)
    {
        frameSize.height = sender.bounds.size.height;
        if (delta &gt; 0 ||
            frameSize.width + delta &gt;= minLengthValue)
        {
            frameSize.width += delta;
            delta = 0;
        }
        else if (delta &lt; 0)
        {
            delta += frameSize.width - minLengthValue;
            frameSize.width = minLengthValue;
        }
    }
    else
    {
        frameSize.width = sender.bounds.size.width;
        if (delta &gt; 0 ||
            frameSize.height + delta &gt;= minLengthValue)
        {
            frameSize.height += delta;
            delta = 0;
        }
        else if (delta &lt; 0)
        {
            delta += frameSize.height - minLengthValue;
            frameSize.height = minLengthValue;
        }
    }

    [view setFrameSize:frameSize];
    viewCountCheck++;
}
</pre>

`continue`에 의해 유효하지 않은 우선순위들은 생략된다. 자잘한 에러들을 어떻게 다룰 것인지에 따라 이 부분을 적절히 수정하여 사용하도록하자.

`delta`가 완전히 적용되었더라도, 루프를 빠져나오지 않는 점에 주의하자. 이는 여전히 각 뷰에 대해`setFrameSize:`를 호출하여 수직하는 방향에대한 크기 변화를 적용해야 할 필요가 있기 때문이다(수직 스플릿뷰라면 column의 높이를, 수평 스플릿뷰라면 row의 너비).

위의 코드에는 나와있지 않지만 예제 프로젝트를 보면 새로운 크기에따라 origin 위치 값을 정확히 재설정하기 위해 모든 뷰에 대해서 한번 더 이터레이션이 적용된다.

## 모든 뷰가 최소 크기에 도달 할 경우의 동작

모든 뷰가 최소크기에 도달하여 더이상 줄어들 수 없을경우, 현재 예제의 구현에서는 assert에 의해 프로그램이 종료된다. 스플릿뷰를 포함하고있는 상위 윈도우 혹은 부모뷰가 특정크기 이하로 줄어드는 것을 방지하고자 함인데, 이 동작을 원하지 않는다면 `setMinimumLength:forViewAtIndex:` 메서드 안의 `NSAssert3` 구문을 삭제하면된다.(역자 주: 최소크기이하로 윈도우가 줄어드는 것 자체를 방지하기 위해서는 인터페이스 빌더에서 해당 윈도우를 선택 후 Minimum Size에대한 Constraint를 활성화 하면 된다.)

## 결론

> PrioritySplitViewDelegate 클래스를 포함한 샘플프로젝트는 다음 링크에서 다운로드 가능하다. [ColumnSplitView.zip (60kb)][5]

PrioritySplitViewDelegate는 `NSSplitView`를 사용할 경우 매번 발생하는 상황을 다루고 있는 generic한 클래스이기 때문에 언제든 재사용이 가능하다.  
`setMinimumLength:forViewAtIndex:`와 `setPriority:forViewAtIndex:`의 코드를 수정해서 잘못된 인덱스나 우선순위 값들을 체크 할 수 있도록 좀더 정교하게 만들 수도 있을 것이다. 하지만 대부분의 크리티컬한 에러들은 `splitView:resizeSubviewsWithOldSize:`코드 안의 `NSAsserts`에 의해 이미 체크되고있다.

### 출처 및 저작권 관련 정보

이 글의 저작권은 [CocoaWithLove.com][6]을 운영중인 Matt Gallagher 에게 있고, 저자의 동의하에 한국어로 번역되었습니다. 원본 글은 다음 링크를 통해 접근 가능합니다.  
[Link to original article][7]

 [1]: /uploads/2013/08/proportionalsmall.png
 [2]: /uploads/2013/08/proportionallarge.png
 [3]: /uploads/2013/08/prioritysmall.png
 [4]: /uploads/2013/08/prioritylarge.png
 [5]: /uploads/2013/08/ColumnSplitView.zip
 [6]: http://www.cocoawithlove.com
 [7]: http://www.cocoawithlove.com/2009/09/nssplitview-delegate-for-priority-based.html