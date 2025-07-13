---
title: '자바스크립트로 크롤러 만들기: 크롤링 개념 및 튜토리얼 소개'
date: 2022-05-09T13:51:37+00:00
url: /javascript-crawler-tutorial-intro/
featured_image: /uploads/2022/05/javascript-crawler-tutorial-og-00-150x79.png
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part2/"     class="post-1014"><span class="crp_title">자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part2/"     class="post-1014"><span class="crp_title">자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - Javascript

---
본 튜토리얼 시리즈에서는 필자가 개발하고 운영했던 코로나19 통계 정보 제공 사이트인 [코로나보드][1]에 실제 사용되었던 코드 기반으로 자바스크립트 크롤러를 만드는 방법에 대해서 설명합니다. 본 글의 내용은 필자가 집필한 [&#8216;코로나보드로 배우는 실전 웹 서비스 개발&#8217;][2] 책의 일부를 발췌해서 튜토리얼 형식에 맞게 재구성하였습니다.

글은 지금 보고계신 소개글 외에 총 4편으로 구성되어있습니다.

  * [자바스크립트로 크롤러 만들기 소개: 크롤링 개념 및 튜토리얼 소개][3]
  * [자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기][4]
  * [자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기][5]
  * [자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기][6]
  * [자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기][7]

그럼 이제 본격적으로 크롤러를 개발하기 전에 개념적인 부분들 부터 소개해보도록 하겠습니다.

### 크롤링이란? {#toc_1}

인터넷에는 수많은 웹페이지가 존재합니다. 사용자는 웹 브라우저를 이용하여 원하는 페이지를 열고, 웹 브라우저에 렌더링<sup>rendering</sup>된 내용을 읽어서 정보를 얻습니다. 한편, 컴퓨터 프로그램을 작성하여 자동화된 방법으로 웹페이지 전체 또는 일부를 추출하여 정보를 얻을 수도 있는데, 이러한 작업을 크롤링 또는 스크래핑<sup>scraping</sup>이라 합니다. 여기서는 크롤링으로 용어를 통일하겠습니다.

### 크롤러의 필요성 {#toc_2}

자주 찾는 데이터를 잘 정리해 알려주는 공식 API가 제공된다면 웹페이지를 따로 크롤링할 필요가 없습니다. 하지만 원하는 데이터가 웹페이지에만 존재할 뿐 API로 제공되지 않는다면 어쩔 수 없이 크롤링을 고려하게 됩니다. 그리고 크롤러를 이용하여 웹사이트의 데이터를 주기적으로 수집할 수 있다면 데이터를 모아서 코로나보드처럼 새로운 가치를 제공하는 서비스를 만들어낼 수 있습니다.

### 크롤링시 주의할 사항 {#toc_3}

무분별한 크롤링으로 웹사이트 제공자에게 과도한 트래픽을 유발하지만 않는다면, 사이트 제공자 입장에서는 크롤러와 일반 사용자가 별 차이가 없으므로 큰 문제가 되지 않습니다.

하지만 웹사이트에서 개인정보 같은 민감한 정보가 포함된 경우라든가, 정보 제공자가 많은 노력을 쏟아서 저작권이 인정되는 데이터 경우, 혹은 정보 자체가 유료인 경우도 있을 수 있습니다. 이런 정보는 함부로 크롤링하여 다른 곳에 이용하면 법적 문제가 발생할 수 있습니다. 그러니 크롤링할 때는 항상 대상 사이트를 크롤링해도 문제될 소지가 없는지를 확인해야 합니다.

## 튜토리얼 진행 순서 안내 {#toc_4}

그러면 이제 본격적으로 크롤링을 하기 위해 필요한 지식들을 어떻게 배워나갈지 단계별로 소개해보겠습니다.

먼저 웹페이지를 크롤링하기 위해서는 먼저 해당 웹페이지가 어떻게 구성되어있는지를 확인할 수 있는 도구를 다룰 수 있어야합니다. 이를 위해 1편에서 크롬 [개발자 도구]를 사용하는 방법에 대해 설명합니다.

개발자 도구를 통해 구성을 확인하고나면 해당 웹페이지에서 원하는 부분만 추출할 수 있어야하는데 이를 위해서는 몇가지 HTML/CSS/JS 배경지식이 필요합니다. 이를 2편에서 알아봅니다.

웹페이지의 구성이나 데이터를 불러오는 방식에 따라 크롤러가 다르게 만들어져야하는데요, 3편에서는 이러한 웹페이지를 유형별로 나눠서 해당 유형에 맞는 크롤링 방법을 제시합니다.

4편에서는 앞서 배운 지식들을 종합하여 예제 웹사이트를 크롤링 해보면서 실제 크롤러를 만들어 봅니다.

이미 크롬 개발자 도구에 익숙하고, DOM이나 CSS 셀렉터에 대해서 잘 알고계신 분들은 1,2편은 생략하고 3편부터 바로 읽으셔도됩니다.

##  {#toc_5}

  * [자바스크립트로 크롤러 만들기 소개: 크롤링 개념 및 튜토리얼 소개][3]
  * [자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기][4]
  * [자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기][5]
  * [자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기][6]
  * [자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기][7]

##  {#toc_6}

<div style="text-align: center;">
  <a class="coronaboard-book-banner mobile-only" href="https://bit.ly/3ymH6eK" target="_blank" rel="noopener noreferrer"><img src="https://coronaboard.kr/images/square-banner.png" alt="코로나보드로 배우는 실전 웹 서비스 개발" style="width: 320px; height:306px;" /></a>
</div>

[&#8216;코로나보드로 배우는 실전 웹 서비스 개발&#8217;][2] 책에서는 크롤러 뿐만 아니라 구글 스프레드 시트를 저장소로 사용하는 방법, 개츠비 + 리액트를 사용해서 웹사이트를 개발하는 방법, 도메인 설정, 검색엔진 최적화, 사용자 분석, 광고를 통한 수익화 등을 다룹니다. 웹서비스 개발과 운영에 대한 전반적인 내용을 알고싶으신 분들이나, 사이드 프로젝트를 만들어보고 싶으신 분들에게 추천합니다.

 [1]: https://coronaboard.kr
 [2]: https://bit.ly/3ymH6eK
 [3]: https://www.letmecompile.com/javascript-crawler-tutorial-intro/
 [4]: https://www.letmecompile.com/javascript-crawler-tutorial-part1/
 [5]: https://www.letmecompile.com/javascript-crawler-tutorial-part2/
 [6]: https://www.letmecompile.com/javascript-crawler-tutorial-part3/
 [7]: https://www.letmecompile.com/javascript-crawler-tutorial-part4/