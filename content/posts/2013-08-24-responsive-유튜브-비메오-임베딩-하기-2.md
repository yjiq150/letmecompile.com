---
title: Responsive 유튜브(비메오) 임베딩 하기
author: yjiq150
type: post
date: 2013-08-24T10:52:54+00:00
url: /responsive-유튜브-비메오-임베딩-하기-2/
dsq_thread_id:
  - 1637443238
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - 워드프레스 개발
tags:
  - 리스폰시브 웹
  - 리스폰시브 비디오 임베딩
  - responsive video embed
  - responsive youtube embed
  - responsive vimeo embed
  - box model
  - 박스 모델

---
워드프레스로 요즘 유행하는 리스폰시브 테마를 구현하다가 비디오를 임베딩 할일이 생겼다. 유튜브(YouTube)나 비메오(Vimeo)의 경우 보통 `iframe`을 사용해서 로딩되기 때문에 가로세로 길이를 고정하지 않고 CSS만 갖고 동적으로 표현하려니 쉽지 않았다. 그냥 jQuery로 계산해서 처리하려다가 혹시나 해서 구글링 해보다가 <http://avexdesigns.com/responsive-youtube-embed/> 글을 발견했다. 계산식도 없이 CSS만 갖고 리사이징을 하는 구현 아이디어를 보니 무릎이 탁 쳐진다. 역시 세상에 참 똑똑하신 분들 많은걸 새삼 깨닫는다.

### HTML 코드

<pre><div class="video-container"> </div>
</pre>

### CSS 코드

<pre>.video-container {
    position: relative;
    padding-bottom: 56.25%;
    padding-top: 30px; height: 0; overflow: hidden;
}

.video-container iframe,
.video-container object,
.video-container embed {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
</pre>

### 동작 원리

  1. `iframe`에 `width:100%`, `height:100%` 속성을 줘서 일단 부모 엘리먼트를 꽉채우도록 만들어 둔다.

  2. 부모(video-container) 엘리먼트가 고정된 `height` 값을 갖고있다면 손쉽게 자식 엘리먼트인 `iframe`의 `height`도 퍼센트 계산을 통해 실제적인 높이값이 계산되지만, 리스폰시브한 영상을 사이징을 위해서 `height` 고정값으로 셋팅하면 안되는 상황이다. 부모엘리먼트가 고정 `height`를 지니지 않은 경우, 자식엘리먼트가 `height`를 100%로 셋팅해봐야 계산할 부모의 기준 값이 없기 때문에 제대로 사이징이 되지 않는 문제가 생긴다.

  3. 결국 이를 해결하기 위해, `width`에 따라 `height`길이가 유동적으로 조절되도록 `padding-bottom: 56.25%` 코드를 넣는것이 핵심이다. 보여주고 싶은 영상비율에 따라 적당히 `width`, `height`에 대한 비율을 퍼센트값으로 `padding-bottom`에 설정해주는 것이다.

  4. 이때 그림에서 보듯이 video-container자체의 내부 컨텐츠 높이는 0px이고, 패딩이 572px 이다. video-container안에 포함된 `iframe`이 video-container 내부 컨텐츠 height의 100%(= 0px)가 아닌 전체 박스모델 height(margin, border, padding, content 모두 포함)의 100%(= 30 + 572px) 로 확장되어야 한다. 이를 위해 `position: absolute` 속성을 추가한다.

[<img loading="lazy" width="321" height="214" src="/uploads/2013/08/Screen-Shot-2013-08-24-at-오후-7.09.42.png" alt="Box model"  class="alignnone size-full wp-image-167" />][1]

결국 한마디로, absolute 포지셔닝과 박스모델 원리를 잘이용한 트릭같은 것인데 리스폰시브 웹페이지 구현을 위해 나중에 써먹을 구석이 많이 있지 않을까 생각된다.

 [1]: /uploads/2013/08/Screen-Shot-2013-08-24-at-오후-7.09.42.png