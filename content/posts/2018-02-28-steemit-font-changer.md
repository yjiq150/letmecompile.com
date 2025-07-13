---
title: 스팀잇 폰트체인저 – 한글 폰트 최적화로 스팀잇 포스트의 가독성을 향상시키기
date: 2018-02-28T00:46:17+00:00
url: /steemit-font-changer/
dsq_thread_id:
  - 6509795053
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
tags:
  - steem steemit 스팀잇폰트

---
스팀잇 사이트의 경우 영문 포스트는 그나마 이쁘게 설정되어있는데, 한글의 경우 폰트 설정도 엉망이고 줄간격, 자간 등이 전혀 한글 읽기에 최적화가 되어있지 않습니다. 특히 윈도우 환경에서 볼때 포스트 본문에 대해 OS 기본 폰트가 로딩되지 않도록 개발이 되어있어서 정말 안타까웠습니다. 그래서 결국 답답한 마음에 크롬 브라우저 사용자들을 위해서 크롬 확장 프로그램을 개발해 보았습니다.

### 스팀잇 폰트 체인저 소개

&#8220;스팀잇 폰트 체인저&#8221; 크롬 확장 프로그램을 설치하면 무엇이 바뀌는지는 다음 스크린샷을 참고해주시면 한눈에 이해하실 수 있습니다.

![스팀잇 폰트 체인저 적용전/적용 후][1] 

조금 더 구체적으로 지원하는 기능들을 나열하면 아래와 같습니다.

  * 한글 읽기에 최적화된 폰트 크기와 자간/줄간격 자동 설정
  * OS 기본 한글 폰트 적용
  * OS 기본 폰트가 마음에 들지 않는다면 나눔고딕/ 나눔명조 무료 웹폰트를 설정해서 사용 가능

![스팀잇 폰트 체인저 주요 기능][2] 

### 스팀잇 폰트 체인저 설치 하기

크롬 브라우저를 사용하신다면 아래 링크를 통해서 바로 크롬 웹스토어로 이동하신 후, 설치해서 사용하시면 됩니다. (안타깝게도 인터넷 익스플로러, 파이어폭스, 사파리는 지원하지 않습니다.)

[스팀잇 폰트체인저 (크롬확장프로그램) 바로가기][3]

확장 프로그램을 설치하면 자동으로 가독성 향상 모드가 적용되면서 폰트, 자간, 줄간격을 자동으로 설정해줍니다. 설치 후에 다시 스팀잇 페이지를 열어보면 스타일이 바뀌어 있는 것을 확인하실 수 있습니다. (이미 열려있던 창은 닫고 다시 열거나 새로고침 해주세요)

해당 모드를 비활성화 하고싶으시거나 추가로 한글 폰트를 적용하고 싶으시다면 익스텐션 아이콘을 클릭해서 설정 팝업창을 띄운 후 옵션을 변경해주시면 됩니다.

### 익스텐션을 설치하면 개인 정보가 유출된다거나 하는 보안 문제는 없나요?

&#8220;스팀잇 폰트체인저&#8221;는 스팀잇 페이지의 폰트 스타일을 강제로 변경해야하기 때문에 크롬익스텐션 설치시에 &#8220;방문하는 웹사이트의 전체 데이터 조회 및 변경&#8221; 에 대한 권한 확인창이 뜹니다. 문구만 읽어보면 좀 무섭긴 하지만 실제로는 steemit.com 사이트에 대해서 폰트 스타일을 변경하는것 외에는 전혀 하는것이 없으니 안심하시고 설치하셔도 됩니다.

추가로 &#8220;인터넷 사용기록 확인&#8221; 권한도 요구합니다. 기술적인 이야기를 좀 하자면 steemit.com의 경우 SPA(Single Page Application) 방식으로 개발되어있어서, 사이트 내에서 링크를 클릭해서 이동할때 전체 페이지가 릴로드 되지 않고 부분적으로 릴로드 한 후 URL만 변경하가있습니다. 이 시점을 디텍트해서 &#8220;스팀잇 폰트체인저&#8221;가 스타일을 업데이트해줘야 하다 보니 URL 변경을 모니터링 하게되었고, 이때문에 해당 권한이 필요해졌습니다. 저도 익스텐션이 사용하는 권한을 최소화 하고 싶었지만 어쩔 수 없이 추가하게 되었으니 이해 부탁드립니다.

그래도 보안은 항상 중요하니 돌다리를 두들겨보고 건너보실 분들이 계실것 같아서 본 익스텐션의 소스 코드를 완전히 공개해 두었습니다.

[스팀잇 폰트체인저 소스 코드 확인][4]

### 마무리

크롬 익스텐션을 활용하면 웹사이트의 스타일을 바꾼다거나, 레이아웃을 변경한다거나, UI 컴포넌트를 추가한다거나 하는 일들을 다양하게 할 수 있습니다. 혹시 더 필요하신 기능이나 개선점이 있다면 피드백 주시면 틈나는대로 업데이트 해보도록 하겠습니다.

 [1]: https://steemitimages.com/DQmRvZTyfLVkDuyGMASPLLjTiygQtTC8Cfc8bvVxAZ9wEVQ/steemit-font-changer-screenshot1.png
 [2]: https://steemitimages.com/DQmeaqP9YmihjSuNNMeW4FK8K2CBo4gLfTHzWpEpKQMCjxe/steemit-font-changer-screenshot2.png
 [3]: https://chrome.google.com/webstore/detail/steemitcom-%ED%8F%B0%ED%8A%B8-%EC%B2%B4%EC%9D%B8%EC%A0%80/gepddagaajclednflogimfmpoejjkcnj
 [4]: https://github.com/yjiq150/steemit-font-changer