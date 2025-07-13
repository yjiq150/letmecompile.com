---
title: NSSplitView 다중 컬럼 생성
date: 2013-07-25T16:33:47+00:00
url: /nssplitview-다중-컬럼-생성/
dsq_thread_id:
  - 1530970169
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/ec2-snapshot-%ec%a3%bc%ea%b8%b0%ec%a0%81%ec%9c%bc%eb%a1%9c-%eb%b0%b1%ec%97%85%ed%95%98%ea%b8%b0/"     class="post-830"><span class="crp_title">EC2 Snapshot 주기적으로 백업하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/ec2-snapshot-%ec%a3%bc%ea%b8%b0%ec%a0%81%ec%9c%bc%eb%a1%9c-%eb%b0%b1%ec%97%85%ed%95%98%ea%b8%b0/"     class="post-830"><span class="crp_title">EC2 Snapshot 주기적으로 백업하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
tags:
  - NSSplitView
  - 다중컬럼
  - NSSplitView multiple column
  - NSSplitView multiple row
  - NSSplitView 나누기
  - NSSplitView 다중 컬럼
  - 코코아
  - cocoa
  - 인터페이스 빌더

---
현재 XMLRPC를 이용해서 워드프레스로 리모트 퍼블리싱 하는 맥용 코코아 어플리케이션을 개발중이다. 등록된 계정/글목록/세부내용을 보여주기위해서 맥에서 자주사용되는 애플 기본 메일앱이나 에버노트, Ulysses 등에서 사용되는 3 column 형태로 개발을 진행하는 중이다.

기본적으로 인터페이스 빌더(Interface Builder)에서 `NSSplitView`를 추가하면 행 또는 열이 2개로 나누어져서 추가된다. 여기서 삽질이 시작되었다. 컬럼 수를 늘리기 위해 IB의 스플릿뷰에대한 옵션을 아무리 찾아봐도 없어서 스플릿뷰의 한쪽에 다시 스플릿뷰를 넣어서 컬럼을 3개로 만들었는데, 아무리 생각해봐도 뭔가 이건 아니다 싶은 느낌이 들었다. 결국 한참을 구글링 한 끝에 IB의 Object Tree 화면에 그냥 `NSView`를 &#8216;Split View&#8217; 항목 아래로 끌어다 놓으면 된다는 사실을 발견하고나서 허탈해서 이 글을 포스팅 한다. &#8216;Split View&#8217; 아래의 subview들 숫자만큼 자동으로 컬럼 수가 나누어질지 누가 알았나 허허. (아래 그림 참조)

[<img loading="lazy" width="940" height="531" src="/uploads/2013/07/nssplitview_multiple.png" alt="NSSplitView multiple column screen shot" class="alignnone size-full wp-image-109" />][1]

컬럼뷰 리사이징에 적절한 비율로 컨트롤하는 것에 대해서는 Matt Gallagher 님이 쓴 다음 글이 있는데 [An NSSplitView delegate for priority based resizing][2] 시간나는대로 번역을 해서 올릴 예정이다.

며칠전에 Matt에게 컨택해서 번역해서 포스팅 해도 된다는 허락을 맡았으니 좋은 글들은 틈틈히 계속 번역할 예정이다. Matt이 지난 2년간 일이 바빠서 글을 거의 안올렸는데, 곧 다시 포스팅을 시작할거라 하니 기대해도 될듯.

 [1]: /uploads/2013/07/nssplitview_multiple.png
 [2]: http://www.cocoawithlove.com/2009/09/nssplitview-delegate-for-priority-based.html