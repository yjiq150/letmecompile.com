---
title: '자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기'
author: yjiq150
type: post
date: 2022-05-09T14:25:06+00:00
url: /javascript-crawler-tutorial-part1/
featured_image: http://www.letmecompile.com/wp/wp-content/uploads/2022/05/javascript-crawler-tutorial-og-01-150x79.png
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part2/"     class="post-1014"><span class="crp_title">자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-intro/"     class="post-1007"><span class="crp_title">자바스크립트로 크롤러 만들기: 크롤링 개념 및 튜토리얼 소개</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li><li><a href="https://www.letmecompile.com/xcode-%eb%8b%a8%ec%b6%95%ed%82%a4-%eb%aa%a8%ec%9d%8c/"     class="post-860"><span class="crp_title">자주쓰는 Xcode 단축키 모음</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part2/"     class="post-1014"><span class="crp_title">자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-intro/"     class="post-1007"><span class="crp_title">자바스크립트로 크롤러 만들기: 크롤링 개념 및 튜토리얼 소개</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li><li><a href="https://www.letmecompile.com/xcode-%eb%8b%a8%ec%b6%95%ed%82%a4-%eb%aa%a8%ec%9d%8c/"     class="post-860"><span class="crp_title">자주쓰는 Xcode 단축키 모음</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - Javascript

---
크롤링을 하기 전에 대상 웹페이지의 구조를 파악하고 원하는 정보만 효율적으로 추출해올 방법을 구상해야 합니다. 웹페이지 구조를 분석하는 가장 좋은 방법은 웹 브라우저에 내장된 [개발자 도구]를 이용하는 겁니다.

[개발자 도구]에서는 현재 웹페이지의 HTML 구조를 라이브로 볼 수 있고, HTML 요소에 적용된 CSS 스타일을 조사한다거나, 웹페이지에서 수행되는 HTTP 요청/응답 내용을 모니터링하거나, 자바스크립트 코드를 디버깅하는 등 매우 다양한 일을 할 수 있습니다.

이러한 기능들은 크롤링할 웹페이지를 파악하는 데도 매우 유용하지만 자신이 개발 중인 웹페이지를 디버깅하는 데도 거의 필수로 쓰이니 사용법을 잘 익혀두면 많은 도움이 됩니다. 이번 장뿐만 아니라 나중에 프런트엔드 개발을 진행할 때도 [개발자 도구]를 계속 사용할 것이라서 일반적으로 자주 사용되는 기능을 하나하나 설명하겠습니다.

이번 절에서는 웹 브라우저 중 가장 많이 사용되는 크롬을 기준으로 설명하겠습니다. 파이어폭스, 사파리, 엣지 <sup></sup>등 다른 웹 브라우저도 비슷한 기능의 [개발자 도구]를 내장하고 있으니 어렵지 않게 적응할 수 있을 겁니다.

크롬 화면에서 마우스 오른쪽 버튼을 눌러 나오는 팝업 메뉴에서 [검사]를 선택하거나, 키보드의 F12 키(맥에서는 Fn + F12)를 누르면 [그림 4-1]과 같은 모습의 [개발자 도구]가 열립니다.

<img loading="lazy" width="1024" height="640" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-1_chrome-developer-tool-overview-1024x640.png" alt="[그림 4-1] 크롬 개발자 도구" class="wp-image-987" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-1_chrome-developer-tool-overview-1024x640.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-1_chrome-developer-tool-overview-300x188.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-1_chrome-developer-tool-overview-768x480.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-1_chrome-developer-tool-overview-150x94.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-1_chrome-developer-tool-overview.png 1340w" sizes="(max-width: 1024px) 100vw, 1024px" /> 

보다시피 화면 왼쪽 ❶에는 웹페이지가 유지되고, 오른쪽 ❷에 [개발자 도구] UI가 나타납니다. 이제부터 ❸ [개발자 도구] 상단 메뉴들을 왼쪽부터 하나씩 설명하겠습니다.

### 요소 선택 커서 토글 {#toc_1}

❸ 상단 메뉴 가장 왼쪽의 사각형과 커서가 같이있는 아이콘을 클릭하면 아이콘 색이 파랗게 바뀝니다. 이 상태로 ❶의 웹페이지 화면 위로 마우스 커서를 옮기면 마우스 커서가 위치한 곳의 HTML 요소가 하이라이트되면서 자세한 정보가 표시됩니다. 동시에 오른쪽의 ➍ [Elements] 탭은 선택된 요소의 HTML 코드를 하이라이트해서 보여줍니다.

이 기능을 사용하면 웹페이지 화면에서 원하는 요소를 빠르게 찾을 수 있으니 꼭 기억해둡시다.

<img loading="lazy" width="992" height="730" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-2_chrome-developer-tool-inspector-highlight.png" alt="[그림 4-2] 요소를 선택해 하이라이트된 모습" class="wp-image-986" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-2_chrome-developer-tool-inspector-highlight.png 992w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-2_chrome-developer-tool-inspector-highlight-300x221.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-2_chrome-developer-tool-inspector-highlight-768x565.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-2_chrome-developer-tool-inspector-highlight-150x110.png 150w" sizes="(max-width: 992px) 100vw, 992px" /> 

### Device Toolbar 토글 {#toc_2}

상단 메뉴에서 두 번째 스마트폰/태블릿을 겹쳐둔 모양의 아이콘은 Device Toolbar를 보이게 하거나 사라지게 합니다. Device Toolbar로는 웹페이지 화면의 해상도를 자유롭게 변경할 수 있어서 해상도에 따라 디자인이 바뀌는 반응형 웹페이지를 손쉽게 테스트할 수 있습니다. 주요 스마트폰과 태블릿의 해상도 프리셋을 제공하기 때문에 스마트폰에서 웹페이지가 어떻게 보이는지를 PC에서도 확인할 수 있습니다. PC와 모바일을 모두 지원하는 웹사이트를 만들 때 자주 사용하는 기능입니다.

<img loading="lazy" width="1024" height="797" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-3_chrome-developer-tool-device-toolbar-1024x797.png" alt="[그림 4-3] Device Toolbar가 활성화된 상태" class="wp-image-985" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-3_chrome-developer-tool-device-toolbar-1024x797.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-3_chrome-developer-tool-device-toolbar-300x234.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-3_chrome-developer-tool-device-toolbar-768x598.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-3_chrome-developer-tool-device-toolbar-150x117.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-3_chrome-developer-tool-device-toolbar.png 1164w" sizes="(max-width: 1024px) 100vw, 1024px" /> 

### Elements 탭 {#toc_3}

[Elements] 탭의 왼쪽 ❶ 영역에서는 현재 웹페이지의 HTML 요소들이 어떻게 구성되었는지 한눈에 확인할 수 있습니다. 관심 있는 요소에서 마우스를 우클릭하여 요소를 추가/수정할 수도 있습니다. 수정 내용은 왼쪽 웹페이지에 실시간 반영됩니다.

오른쪽의 [Styles] 탭에서는 선택된 요소에 적용된 CSS 정보를 확인할 수 있습니다.

❷ 영역을 클릭하면 새롭게 CSS 속성들을 추가로 작성하여 새로운 속성을 정의하거나 기존에 설정된 속성들을 오버라이드<sup>override</sup>할 수도 있습니다. 웹페이지에서 외부 CSS 파일 등을 로드한 경우 해당 파일에 정의된 속성값들이 요소에 적용된 경우가 많이 있는데요, ❸ 영역에서는 해당 요소에 적용된 CSS 속성을 확인할 수 있고 적용된 속성들을 자유롭게 수정할 수 있습니다. ❷, ❸ 영역에서 속성을 변경하면서 웹페이지에 디자인이 반영되는 것을 실시간으로 볼 수 있기 때문에 디자이너와 개발자 모두에게 매우 유용한 기능입니다.

참고로 [Elements] 탭에서 수정한 내용은 페이지를 새로고침하면 원래 웹페이지로 돌아가게 됩니다. 그러니 마음 놓고 값을 바꿔가면서 테스트해도 아무런 문제가 없습니다.

<img loading="lazy" width="1024" height="727" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-4_chrome-developer-tool-element-tab-1024x727.png" alt="[그림 4-4] [Elements] 탭 사용하기" class="wp-image-984" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-4_chrome-developer-tool-element-tab-1024x727.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-4_chrome-developer-tool-element-tab-300x213.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-4_chrome-developer-tool-element-tab-768x545.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-4_chrome-developer-tool-element-tab-150x107.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-4_chrome-developer-tool-element-tab.png 1104w" sizes="(max-width: 1024px) 100vw, 1024px" /> 

### Console 탭 {#toc_4}

[Console] 탭에서는 자바스크립트에서 console.log, console.error 함수 등을 이용하여 출력하는 로그를 확인할 수 있습니다. 이 외에도 네트워크 요청 오류가 발생하거나 자바스크립트 코드 실행 중에 예외가 발생하면 오류 메시지를 출력해줍니다. 따라서 웹페이지가 의도대로 동작하지 않는다면 콘솔을 열어서 오류 메시지가 있는지 확인하는 것이 좋습니다.

참고로 [개발자 도구]가 켜져 있는 상태에서 선택된 탭과 무관하게 ESC 키를 누르면 아래쪽에 [Console] 영역이 나타났다 사라졌다 합니다. 이렇듯 [Console]은 아래쪽에 항상 별도로 나타나게 할 수 있으므로 굳이 상단 메뉴의 [Console] 탭을 선택하여 보는 경우가 거의 없긴 합니다.

콘솔창에서는 간단한 자바스크립트 코드를 실행해볼 수도 있습니다. 다음처럼 ‘윈도우’라고 코드를 입력하고 Enter키를 쳐서 실행하면 윈도우 객체의 요약 정보가 출력됩니다(윈도우는 웹 브라우저에 항상 존재하는 전역 객체입니다). 왼쪽 부분의 삼각형 아이콘을 클릭하면 해당 객체에 대한 더 자세한 정보를 살펴볼 수 있습니다.

<img loading="lazy" width="1024" height="290" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-5_chrome-developer-tool-console-1024x290.png" alt="[그림 4-5] 콘솔창에 코드를 입력하여 실행하기" class="wp-image-983" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-5_chrome-developer-tool-console-1024x290.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-5_chrome-developer-tool-console-300x85.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-5_chrome-developer-tool-console-768x218.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-5_chrome-developer-tool-console-150x43.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-5_chrome-developer-tool-console.png 1094w" sizes="(max-width: 1024px) 100vw, 1024px" /> 

입력된 코드는 현재 웹페이지에 포함된 자바스크립트 코드가 모두 로드된 상태에서 실행됩니다. 즉, 현재 페이지의 자바스크립트 변수에 의도한 값이 잘 들어 있는지 확인해볼 수 있습니다. 또한 코드를 입력하면 바로바로 실행되기 때문에, 잘 이용하면 개발 생산성 향상에도 큰 도움을 줄 수 있습니다.

### Sources 탭 {#toc_5}

현재 페이지에 로드된 다양한 종류의 소스 파일을 한눈에 확인할 수 있습니다. [Page] 탭을 보면 자바스크립트 파일(.js), CSS 파일(.css) 등 다양한 파일이 어떤 호스트로부터 로드되었는지 알 수 있고, 각 파일을 클릭하면 그 내용이 오른쪽 코드 영역에 나타납니다.

특히 자바스크립트 파일을 열어서 줄번호 영역을 클릭하면 브레이크포인트<sup>break point</sup>가 지정됩니다. 이 기능을 이용하면 웹 브라우저에서 자바스크립트 소스 코드를 보면서 라이브로 디버깅이 가능하여 매우 유용합니다.

> **용어: 브레이크포인트**
> 
> 코드의 특정 위치에 브레이크포인트를 지정하면 프로그램이 실행되다가 해당 위치에 도달하는 순간 일시적으로 실행을 멈추게됩니다. 이렇게 프로그램의 실행을 일시적으로 멈춰놓으면 실행중이던 프로그램에 존재하는 다양한 변수들에 저장된 값들을 손쉽게 확인할 수 있습니다. 이를 통해 프로그램이 자신이 의도한 대로 동작하지 않는 경우 적절한 위치에 브레이크포인트를 지정하여 원인 파악을 하기위해 많이 사용합니다.

<img loading="lazy" width="1024" height="561" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-6_chrome-developer-tool-breakpoint-1024x561.png" alt="[그림 4-6] [Sources] 탭에서 브레이크포인트 설정하기" class="wp-image-982" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-6_chrome-developer-tool-breakpoint-1024x561.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-6_chrome-developer-tool-breakpoint-300x164.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-6_chrome-developer-tool-breakpoint-768x421.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-6_chrome-developer-tool-breakpoint-150x82.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-6_chrome-developer-tool-breakpoint.png 1222w" sizes="(max-width: 1024px) 100vw, 1024px" /> 

자바스크립트는 파일 크기를 최소화하기 위해 소스 코드를 미니파이 및 어글리파이해서 배포하기도 합니다. 그러면 불필요하게 긴 변수 이름을 짧게 줄이고 공백, 세미콜론, 줄바꿈 문자까지 모두 삭제되기 때문에 [그림 4-7]처럼 한 줄로 표현됩니다. 코드가 한 줄이면 읽기도 힘들 뿐더러 디버깅을 위해 브레이크포인트를 지정하는 것은 더욱 힘이 듭니다. 이럴 때는 [pretty print] 버튼을 클릭하면 그나마 코드 형태를 사람이 읽기 좋은 형태로 바꿔주기 때문에 매우 유용합니다..

> **용어: 미니파이(minify), 어글리파이(uglify)**
> 
> 미니파이는 코드 실행에 불필요한 주석, 공백, 세미콜론, 줄바꿈 문자들을 삭제해 용량을 줄이는 기법입니다. 어글리파이는 미니파이에서 한 단계 더 나아가 변수 이름이나 함수 이름 등을 가능한 짧게 대체하여 용량을 줄입니다. 어글리파이된 코드는 사람이 분석하기 어렵습니다.

<img loading="lazy" width="898" height="894" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-7_chrome-developer-tool-pretty-print.png" alt="[그림 4-7] [Sources] 탭에서 미니파이된 코드를 pretty print하기" class="wp-image-981" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-7_chrome-developer-tool-pretty-print.png 898w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-7_chrome-developer-tool-pretty-print-300x300.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-7_chrome-developer-tool-pretty-print-150x149.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-7_chrome-developer-tool-pretty-print-768x765.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-7_chrome-developer-tool-pretty-print-50x50.png 50w" sizes="(max-width: 898px) 100vw, 898px" /> 

### [Network] 탭 {#toc_6}

[Network] 탭은 현재 웹페이지에서 발생하는 모든 HTTP 요청과 응답을 모니터링한 결과를 보여줍니다.

<img loading="lazy" width="1024" height="712" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-8_chrome-developer-tool-network-tab-1024x712.png" alt="[그림 4-8] [Network] 탭 살펴보기" class="wp-image-980" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-8_chrome-developer-tool-network-tab-1024x712.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-8_chrome-developer-tool-network-tab-300x209.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-8_chrome-developer-tool-network-tab-768x534.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-8_chrome-developer-tool-network-tab-150x104.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-8_chrome-developer-tool-network-tab.png 1292w" sizes="(max-width: 1024px) 100vw, 1024px" /> 

왼쪽 위 레코딩<sup>recording</sup> 버튼이 빨간색이면 현재 요청/응답을 계속 레코딩한다는 뜻입니다. 그 옆의 [Clear] 버튼을 누르면 레코딩된 내용이 모두 초기화됩니다. ❷ 영역에는 레코딩된 HTTP 요청에 대한 타임라인이 표시됩니다. 이를 통해서 어떤 요청이 응답을 받는 데까지 얼마나 걸리는지, 한꺼번에 얼마나 많은 요청이 병렬로 수행되고 있는지를 한눈에 볼 수 있습니다. ❸ 영역에서는 개별 요청/응답을 하나하나 살펴볼 수 있습니다. 상단 메뉴 중 깔때기 모양의 [Filter] 아이콘을 선택하면 ❶ 영역이 나타납니다. 이 영역에 키워드를 입력하면 레코딩된 결과들을 요청 이름 기준으로 필터링하여 볼 수 있고 [XHR], [JS], [CSS] 등의 버튼을 클릭해 타입별로 필터링할 수도 있습니다.

[Name]에서 특정 요청/응답을 선택하면 오른쪽에 추가로 탭이 나타납니다.

<img loading="lazy" width="1024" height="555" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-9_chrome-developer-tool-request-response-1024x555.png" alt="[그림 4-9] 개별 요청/응답 자세히 살펴보기" class="wp-image-979" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-9_chrome-developer-tool-request-response-1024x555.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-9_chrome-developer-tool-request-response-300x163.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-9_chrome-developer-tool-request-response-768x416.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-9_chrome-developer-tool-request-response-150x81.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-9_chrome-developer-tool-request-response.png 1162w" sizes="(max-width: 1024px) 100vw, 1024px" /> 

이곳의 [Headers] 탭에서는 선택된 HTTP 요청/응답의 헤더 정보가 출력됩니다. 헤더 정보로부터 캐시 지시자, 인증 정보, 쿠키 등, HTTP 본문에는 나타나지 않는 다양한 정보를 알 수 있습니다. [Preview]나 [Response] 탭을 선택하면 실제 HTTP 응답 바디를 확인할 수 있습니다.

많은 웹페이지가 처음 HTML을 로드한 후 자바스크립트 코드를 통해서 추가로 HTTP API 호출을 하여 데이터를 획득한 후 이 데이터를 이용하여 웹페이지를 동적으로 다시 만들어서 사용자에게 보여주는 방식을 사용합니다. HTTP API 호출만 모아서 보고 싶으면 위에서 살펴본 필터 메뉴에서 XHR<sup>XMLHttpRequest</sup>을 선택하면 됩니다. 이 방식을 사용하는 웹페이지에서 우리가 원하는 데이터를 얻으려면 렌더링된 페이지를 크롤링하는 것보다 API를 호출을 모니터링하다가 HTTP 응답 본문을 확인하여 원하는 정보가 포함되었는지 확인할 수 있습니다. 찾아낸 API를 나중에 원하는 시점에 호출하여 바로 정보를 획득하면 웹페이지를 크롤링하는 수고를 덜 수 있습니다.

마지막으로 응답 본문 자체를 검색하는 기능도 존재합니다. 찾고자 하는 응답 내용이 명확한 경우 이 기능을 사용하여 직접 검색할 수 있습니다. 아래 그림에서 ❶의 돋보기 버튼을 클릭하면 왼쪽에 ❷ 영역이 보여집니다. 여기에 검색 키워드를 입력하고 Enter를 누르면 응답 본문에 해당 키워드를 포함하는 요청/응답이 모두 검색되고, 이를 클릭하면 해당 응답 본문이 보여지면서 하이라이트 처리됩니다.

<img loading="lazy" src="http://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-10_chrome-developer-tool-search-body-1024x770.png" alt="[그림 4-10] 응답 본문 검색하기" class="wp-image-978" width="580" height="436" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-10_chrome-developer-tool-search-body-1024x770.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-10_chrome-developer-tool-search-body-300x226.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-10_chrome-developer-tool-search-body-768x577.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-10_chrome-developer-tool-search-body-150x113.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2022/05/4-10_chrome-developer-tool-search-body.png 1200w" sizes="(max-width: 580px) 100vw, 580px" /> 

### 그 외 기능 {#toc_7}

[Performance] 탭은 웹페이지 성능 최적화를 위한 프로파일링 기능을 제공합니다. [Memory] 탭은 힙<sup>heap</sup> 메모리 스냅샷을 찍어서 자바스크립트 코드에서 발생하는 메모리 누수 등을 찾을 때 유용합니다. [Application] 탭은 현재 웹페이지가 웹 애플리케이션 형태로 사용될 때 사용하는 다양한 기능을 살펴볼 수 있습니다. 예를 들어 로컬 스토리지<sup>Local Storage</sup>나 쿠키<sup>cookie</sup>에 들어 있는 값을 확인할 수 있습니다. 이 외에도 많은 기능이 있지만 이 책의 범위를 벗어나기 때문에 자세히 다루지는 않겠습니다.

## 마무리 {#toc_8}

자 이제 크롤링을 위해 필요한 크롬 개발자 도구 사용방법은 모두 익혔습니다. 크롬 개발자 도구는 크롤링이 아니더라도 웹 개발을 할때 필수적인 도구 중 하나이니, 잘 알아두면 여러모로 편리할 것입니다. 다음 2편에서는 웹페이지의 크롤링에 쓰이는 기술을 이해하기 위해 배경 지식을 계속 살펴보도록 하겠습니다.

##  {#toc_9}

  * [자바스크립트로 크롤러 만들기 소개: 크롤링 개념 및 튜토리얼 소개][1]
  * [자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기][2]
  * [자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기][3]
  * [자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기][4]
  * [자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기][5]

##  {#toc_10}

<div style="text-align: center;">
  <a class="coronaboard-book-banner mobile-only" href="https://bit.ly/3ymH6eK" target="_blank" rel="noopener noreferrer"><img src="https://coronaboard.kr/images/square-banner.png" alt="코로나보드로 배우는 실전 웹 서비스 개발" style="width: 320px; height:306px;" /></a>
</div>

[&#8216;코로나보드로 배우는 실전 웹 서비스 개발&#8217;][6] 책에서는 크롤러 뿐만 아니라 구글 스프레드 시트를 저장소로 사용하는 방법, 개츠비 + 리액트를 사용해서 웹사이트를 개발하는 방법, 도메인 설정, 검색엔진 최적화, 사용자 분석, 광고를 통한 수익화 등을 다룹니다. 웹서비스 개발과 운영에 대한 전반적인 내용을 알고싶으신 분들이나, 사이드 프로젝트를 만들어보고 싶으신 분들에게 추천합니다.

 [1]: https://www.letmecompile.com/javascript-crawler-tutorial-intro/
 [2]: https://www.letmecompile.com/javascript-crawler-tutorial-part1/
 [3]: https://www.letmecompile.com/javascript-crawler-tutorial-part2/
 [4]: https://www.letmecompile.com/javascript-crawler-tutorial-part3/
 [5]: https://www.letmecompile.com/javascript-crawler-tutorial-part4/
 [6]: https://bit.ly/3ymH6eK