---
title: 타임존 시차 계산기 앱 ‘리모클(Remocle)’ 추천
date: 2020-11-12T16:13:25+00:00
url: /timezone-conveter-world-clock-app-remocle/
cover:
  image: "/uploads/2020/11/open-graph.jpg"
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 앱추천

---

## 골치아픈 타임존(Timezone) 시차 계산

코로나 대유행이 시작되기 전인 2019년에는 아내와 함께 디지털 노마드 개발자 생활을 하면서 미국 서부와 태국 등을 돌아다녔다. 몸은 해외에 있었지만 국내 회사에 다니면서 리모트 워크를 하고있었기 때문에 계속 한국에 있는 팀원들과 협업을 했고, 때문에 항상 한국 시간과 현지 시간을 신경써야 했다. 특히 미국 서부는 시간대가 PST(Pacific Standard Time), MST(Mountain Standard Time) 두개인데, 타임존 경계 중간쯤 머무르는 날에는 방문하는 목적지에 따라 하루에도 여러번 +1시간 -1시간이 오락가락해서 총 3개의 시간대를 고려해서 시차를 계산해야 해야 했다.

그러다보니 매번 아래와 같은 의문이 하루에도 몇번씩 머릿속에 떠올랐다.

> 한국시간으로 회의가 오후 3시에 잡혀있는데 PST로는 몇시지?  
> 내일 한국시간으로 오전10시에 마케팅 푸시가 예정되어있는데 MST로는 몇시지?

스마트폰에서 기본 제공하는 세계 시계(world clock)의 경우 현재 시간 기준으로만 시차를 보여주다보니, 위 질문과 같이 미래 또는 과거의 시간에 대해서 시간을 알아내려면 계속 머릿속으로 덧셈 뺄셈을 해야했고, 매번 미팅 시간이 언제인지 헷갈리곤 했다.

이런 반복되는 불편함은 개발자인 나와 아내에게 있어 충분히 개선이 가능해 보이는 문제로 보였다. 이런 문제를 해결하기 좋아하는 우리 부부는 고민끝에 리모트 워크 (remote work)를 하는 사람들을 위해 시차 계산을 매번 계산하지않고 눈으로 볼 수 있는 시계 앱을 만들기로 결정했다.

## What is Remocle?

일단 앱을 만들기로 결정을 했으니 리모트 워크를 하는 사람들을 위한 시계를 뭐라고 이름지을까 고민하느라 이것저것 노트에 끄적여 보았다. 그러다가 &#8220;리모트워크 시계(remote work clock)&#8221; 단어 나열이 눈에 들어왔다.

너무 길어서 끄적대면서 좀 줄여봤더니 리모클 이라는 귀여운 이름이 탄생했다.

> 리모트 워크 클락 > 리모트 클락 > 리모클 > Remocle

오오&#8230;. 이거 내가 지은 서비스 이름 맞나? 너무 맘에 들어서 갑자기 대박 예감이 들었다. (과연&#8230;?)

이 글을 더 읽기도 전에 앱이 다운받고 싶어졌다면 [리모클 Remocle][2] 웹사이트로 바로 직행하면 된다.

## 그래서 리모클을 쓰면 뭐가 편한가요?

나랑 아내의 앱 개발 경력을 합치면 10년이 넘는다..! 10년간의 경험을 총동원 해서 타임존 계산의 불편함을 앱으로 해결하기 위해서 매달렸고, 이를 통해 리모클에 아래와 같은 질문에 대한 답을 앱에 담았다.

### 타임존에따라 시간 차이를 눈으로 볼 수 있다면?

인간의 뇌는 항상 단순 숫자만 보면 느낌이 딱 하고 오지 않는다. 그래서 항상 차트나 그래프로 시각화를 했을때 보이지 않던 규칙이 보이고 더 많은 정보를 파악할 수 있게 된다. 그래서 리모클에서는 여러 타임존들을 각각의 막대 그래프로 표시하고 막대 위에 해당 타임존의 시간을 표기했다. 이 막대 위에서 사용자가 타임 커서를 원하는 대로 움직이면 현재 시간 뿐만 아니라 과거/미래 시간까지 볼 수 있게 된다.<figure class="wp-block-image size-large">

<img loading="lazy" width="576" height="1024" class="wp-image-893" src="/uploads/2020/11/screenshot1-576x1024.png" alt="" srcset="/uploads/2020/11/screenshot1-576x1024.png 576w, /uploads/2020/11/screenshot1-169x300.png 169w, /uploads/2020/11/screenshot1-150x267.png 150w, /uploads/2020/11/screenshot1.png 696w" sizes="(max-width: 576px) 100vw, 576px" /> </figure> 

단순히 커서를 움직에서 시간을 보는것 외에도, 막대 위에 파란색으로 지정된 것처럼 사용자가 해당 타임존에 특정 시간대를 표기해 둘 수 있다. 이 기능을 이용하면 세계 각지에서 일하는 사람들과 회의를 해야할 때,겹치는 업무시간을 한눈에 보고 찾을 수 있다. 해외 주식 투자에 관심이 많으신 분들이라면 세계 주요 증시가 언제 개장하고 폐장하는지도 한눈에 알 수 있다. 표시된 영역에 원하는 이모지를 넣어서 꾸밀 수 있는 기능은 덤!<figure class="wp-block-image size-large">

<img class="wp-image-893" src="/uploads/2020/11/screenshot4-576x1024.png" alt="" /> </figure> 

### 잘 모르는 도시의 타임존도 알 수 있을까?

<https://geonames.org> 에서 제공하는 데이터베이스를 기반으로 7000개의 주요 도시 와 해당 도시의 타임존을 제공한다. 미국처럼 땅덩어리가 큰 나라들은 같은 국가에서도 타임존을 다르게 사용하기 때문에 타임존이 헷갈리는 경우가 많은데, 리모클에서 타임존을 알고싶은 도시 이름을 검색해서 추가하면 손쉽게 해당 도시가 사용하는 타임존을 알아낼 수 있다.<figure class="wp-block-image size-large">

<img loading="lazy" width="576" height="1024" class="wp-image-894" src="/uploads/2020/11/screenshot2-576x1024.png" alt="" srcset="/uploads/2020/11/screenshot2-576x1024.png 576w, /uploads/2020/11/screenshot2-169x300.png 169w, /uploads/2020/11/screenshot2-150x267.png 150w, /uploads/2020/11/screenshot2.png 696w" sizes="(max-width: 576px) 100vw, 576px" /> </figure> 

리모클에 대해 더 궁금하면 [리모클 Remocle][2] 웹사이트를 통해서 더 자세히 알아보고 다운로드 하는 것이 가능하다.

## 그래서 이거 공짜인가요?

물론, 무료 다운로드 가능하다! 하지만 광고가 달려있고 쓰다보면 약간의 기능제한이 걸려있다. 인앱결제를 통해 프로 버전을 구입하면 광고 없이 원하는 대로 쓸 수 있다. 꼭 프로버전을 구입하지 않더라도 자주 사용해 주는 것 만으로도 개발자에게는 큰 힘이 되니 잘 부탁드립니다 😀

### 리모클 다운로드 링크

  * [iOS 앱스토어][3]
  * [Android 플레이스토어][4]

 [2]: https://remocle.com/ko
 [3]: https://apps.apple.com/us/app/id1522259532
 [4]: https://play.google.com/store/apps/details?id=com.dundinstudio.remocle
