---
title: 개발자를 위한 필수 맥 앱(Mac App) 10선
author: yjiq150
type: post
date: 2019-02-13T18:31:00+00:00
url: /mac-app-recommendation-for-developer/
featured_image: http://www.letmecompile.com/wp/wp-content/uploads/2019/02/sublime_merge_mac_app-150x104.png
dsq_thread_id:
  - 7230379312
dsq_needs_sync:
  - 1
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/manage-aws-resources-with-pulumi-iac/"     class="post-1058"><span class="crp_title">Pulumi를 이용하여 코드로 AWS 리소스 관리하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/manage-aws-resources-with-pulumi-iac/"     class="post-1058"><span class="crp_title">Pulumi를 이용하여 코드로 AWS 리소스 관리하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 맥 Tips
  - 맥 유틸리티 추천
tags:
  - 맥 앱
  - 맥앱
  - mac
  - app
  - developer
  - tool

---
윈도우에서 열심히 개발을 하다가 맥을 처음 쓰는 개발자라면 맥 환경이 개발자에게 편하다라는 말은 많이 들었을 것이다. 하지만 막상 맥으로 옮기고나서 익숙했던 윈도우용 필수 유틸리티들의 대체품을 빨리 찾지못하면 작업 효율이 오르지 않아 답답할 것이다. 이런 답답함을 풀어드리기 위해 이 포스트를 작성해 보았다. 물론 맥을 원래 부터 쓰고있었지만 새롭게 개발을 시작하는 분들에게도 유용하리라 믿는다.

(덧: 여기서 소개한 앱 외에도 개발 관련해서 좋은 맥 앱들이 있으면 댓글로 추천부탁합니다! 제가 직접 써본 후 글을 업데이트 하겠습니다.)

## HTTP Client

### Postman

터미널에서 curl로 HTTP 요청을 마구 날리면 옆자리 앉은 동료가 보기에 간지나고 좋긴한데 그래도 역시 눈으로 보면서 하면 더 편한 부분이 있지 않을까? 라는 생각이 들 때가 있다. 그렇다면 당신은 GUI HTTP Client인 Postman이 필요한 사람!

<img loading="lazy" class="aligncenter size-large wp-image-844" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/postman_mac_app-1024x820.png" alt="" width="625" height="500" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/postman_mac_app-1024x820.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/postman_mac_app-300x240.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/postman_mac_app-768x615.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/postman_mac_app-150x120.png 150w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://www.getpostman.com/>
  * 기본적인 기능들은 무료이고 workspace 공유 기능이 필요한 경우 유료이다.
  * curl의 훌륭한 대체품이자 보조 수단. 심지어 Postman에서 작성해둔 request를 curl command로 export 해주는 기능이 있다!
  * 환경 변수(environment variable) 기능이 있어서 server phase별 token이나 url 등을 로 저장해두고 요청을 만들때 불러와서 사용할 수 있다. 필자는 local, dev, prd 같이 만들어두고 스위칭해서 사용중이다.

## Web Debugging Proxy

웹 개발자라면 Chrome에 붙어있는 Chrome dev tool이 있으니 손쉽게 http request/response가 오고가는걸 볼 수 있다. 하지만 네이티브 앱 (ex: iOS/Android App)을 개발하고 있다면?

대표적인 웹 디버깅 프록시인 Proxyman 또는 Charles HTTP proxy를 사용해 보자! 개인적으로는 최근에 나온 Proxyman이 무료버전의 기능 제약도 덜하고, 기능 또한 좀더 좋은것 같다. (심지어 protobuf 내용까지 모니터링 할 수 있다)

모바일 디바이스에서 연결된 WiFi 설정화면에서 Proxyman또는 Charles가 실행중인 컴퓨터로 Proxy 설정을 해주면 컴퓨터에서 앱에서 오고가는 모든 HTTP request/response를 손쉽게 확인 할 수 있게된다. HTTPS를 사용하는 경우 바로 되지는 않고 인증서 관련 초기 설정이 좀 어렵긴 하지만 한번 배워두면 생산성이 마구 올라간다.

특히 내가 개발하는 앱 뿐만 아니라 아닌 다른 앱들이 어떤 방식으로 서버와 통신하는지 살펴보면서 공부 할 때도 매우 편리하다.

### Proxyman

[<img loading="lazy" class="aligncenter size-large wp-image-970" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman-1024x635.png" alt="" width="625" height="388" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman-1024x635.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman-300x186.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman-768x477.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman-1536x953.png 1536w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman-2048x1271.png 2048w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman-150x93.png 150w" sizes="(max-width: 625px) 100vw, 625px" />][1]

  * [https://proxyman.io/][2]
  * 유료 앱이고 비싸지만 값어치를 한다!
  * 무료 버전의 경우 Charles와 달리 시간제약은 없고 기능 제약이 존재한다. 기능 제약이 있더라도 기본적인 기능은 무료버전으로도 어느정도 사용이 가능하다.
  * HTTP, HTTPS 모두 지원하고 protobuf 또한 지원한다.
  * rewrite 기능이 있어서 조건에 따라 특정 응답을 마음대로 바꿔치기 할 수 있어 test용도로 매우 유용하다.

### Charles HTTP proxy

<img loading="lazy" class="aligncenter size-large wp-image-842" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/charles_http_proxy-1024x820.png" alt="" width="625" height="500" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/charles_http_proxy-1024x820.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/charles_http_proxy-300x240.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/charles_http_proxy-768x615.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/charles_http_proxy-150x120.png 150w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://www.charlesproxy.com/>

## Git GUI Client

터미널과 CLI를 사랑하는 개발자 라면 git 명령어 만으로도 충분히 살아갈 수 있지만 브랜치가 늘어나고 Merge 커밋이 난무하면서 트리가 복잡해지면 대체 어디서 무슨일이 일어나고 있는지 대략 난감해질 때가 있다. 이럴때 아래 두 맥 앱들 중 하나를 적절하게 선택해서 GUI로 트리 모양을 확인해보면 정신이 맑아진다.

### SourceTree

개발 도구의 명가 Atlasian에서 만든 앱으로 초보자가 보기에 가장 쉽고 편리한 UI를 제공한다. git을 쓴지 얼마 안된 입문자라면 SourceTree를 추천한다.

<img loading="lazy" class="aligncenter size-large wp-image-841" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/SourceTree_mac_app-1024x770.png" alt="" width="625" height="470" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/SourceTree_mac_app-1024x770.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/SourceTree_mac_app-300x226.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/SourceTree_mac_app-768x578.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/SourceTree_mac_app-150x113.png 150w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://www.sourcetreeapp.com/>
  * 무료앱이다!
  * 자주쓰는 일반적인 기능이 잘 노출되어있어서 적응하기에 어렵지 않다.
  * Staged/Unstaged file 섹션이 리스트로 별도 분리되어있어서 파일 변경 내역이 많을 때 파일 변경 리스트/ 파일별 변경 내용을 보기 편하다
  * 여러 커밋들을 동시 선택하면 해당 커밋들에 포함된 모든 변경 내역을 모아서 볼 수 있다.
  * 저장소에 브랜치와 커밋 히스토리가 많아지기 시작하면 점점 느려진다. 메모리 먹는 괴물이 되어서 2GB 넘게 메모리를 차지하고 버벅버벅이는 경우를 많이 봤다.
  * interactive rebase기능이 직관적이어서 이해하긴 쉬운데 좀 불편하다. 코드리뷰 받고 코드 수정할 때 여러 커밋 중 중간에 낀 특정 커밋의 내용을 수정하고 싶을때가 있는데 이게 안된다. (Sublime Merge는 된다!!)

### Sublime Merge

코드 에디터로 유명한 Sublime Text를 만든 곳에서 출시한 앱으로 나온지 채 1년이 되지 않았지만 매우 강력하고 빠른 속도를 자랑한다. 적응하는데 시간이 좀 걸리지만 개인적으로는 적응하고 나니 오히려 SourceTree 보다 월등히 빠른 속도 때문에 훨씬 자주 사용중이다. git 하드코어 유저라면 이쪽을 좀 더 추천한다.

<img loading="lazy" class="aligncenter size-large wp-image-840" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/sublime_merge_mac_app-1024x710.png" alt="" width="625" height="433" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sublime_merge_mac_app-1024x710.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sublime_merge_mac_app-300x208.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sublime_merge_mac_app-768x533.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sublime_merge_mac_app-150x104.png 150w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://www.sublimemerge.com/>
  * 무료 버전에 기능제약은 없지만 가끔 &#8220;구매하실래요?&#8221; 팝업이 뜬다. 추가로 Dark theme을 사용하고싶다면 구매해야한다.
  * 아이콘이 살짝 불명확해서 처음에 좀 적응이 안되지만 무시하고 단축키를 외우면 된다.
  * Cmd + P 로 실행하는 command palette 기능을 사용하면 GUI 앱인데도 불구하고 CLI 비슷하게 사용 가능하다.
  * Staged/Unstaged file에 대한 별도 리스트가 없고 변경된 파일명, 변경 내용이 같이보여서 어떤파일들이 변경되었는지 한눈에 보기가 어렵다.
  * 저장소에 브랜치와 커밋 히스토리가 많아져도 계속 빠르다. 매우 빠르다. 정말 빠르다.
  * 코드리뷰 받고 코드 수정할 때 여러 커밋 중 중간에 낀 특정 커밋의 내용을 수정하는것이 가능하다! 두개의 커밋을 선택해서 바로 스쿼시하는것도 된다. 즉, Edit Commit 기능이 SourceTree에 비해서 월등히 좋으니 이 기능을 많이 이용한다면 Sublime Merge를 쓰면 된다.

## Text Editor

### Visual Studio Code

다들 한번쯤 써보셨으리라 믿고 더이상의 자세한 설명은 생략한다.

### Sublime Text

가볍고 빠르면서도 syntax highlight + 자동완성을 지원한다. 꼭 소스코드 작성이 아니더라도 다양한 텍스트 작업을 할 때 편리하다.

  * <https://www.sublimetext.com/>
  * 무료 버전에 기능제약은 없지만 가끔 &#8220;구매하실래요?&#8221; 팝업이 뜬다.
  * Visual Studio Code나 Atom이 나오기 전에는 다들 이걸 많이썼기 때문에 거의 동급의 기능들을 지원하는 에디터라고 보면 된다.
  * &#8220;Option + 마우스 커서 드래그&#8221; 를 이용해서 으로 커서를 여러줄에 위치시켜두고 동시 편집 하는것이 매우 유용하다. (물론 다른 IDE들도 된다)

### MacDown

Mac에서 마크다운 문서를 작성할 일이 있다면 이 앱이 가장 무난하다. 이 글도 현재 MacDown을 이용해서 작성중이다. (이것이 바로 Bootstrapping!)

<img loading="lazy" class="aligncenter size-large wp-image-843" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/MacDown_mac_app-1024x624.png" alt="" width="625" height="381" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/MacDown_mac_app-1024x624.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/MacDown_mac_app-300x183.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/MacDown_mac_app-768x468.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/MacDown_mac_app-150x91.png 150w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * 무료 앱이다!
  * Live 미리보기 지원
  * Syntax highlight
  * 단축키로 Markdown 문법 적용 가능

## Terminal

### iTerm2

Mac OS에 기본 설치된 terminal도 기본기능은 쓸만하지만 조금 더 하드코어하게 쓰려고 하면 부족한 점이 많이 존재한다. 이때 iTerms2를 사용하면 터미널 생활이 즐거워진다.

<img loading="lazy" class="aligncenter size-large wp-image-845" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/iTerms2_mac_app-1024x592.png" alt="" width="625" height="361" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/iTerms2_mac_app-1024x592.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/iTerms2_mac_app-300x173.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/iTerms2_mac_app-768x444.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/iTerms2_mac_app-150x87.png 150w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://www.iterm2.com/>
  * 무료 앱이다!
  * 터미널 탭/윈도우 분할
  * Window Arrangement Save/Restore 기능 &#8211; 자신이 탭/윈도우 분할을 최적의 상태로 설정 후 Save해두면 컴퓨터가 재부팅되더라도 Restore만 눌러주면 원래 상태로 돌아온다.
  * 명령어 history 제공
  * 단축키, 트랙패드 제스쳐 커스터마이즈
  * 이 외에도 셀수없이 수많은 기능을 지원하니 직접 공부해보시길..

### Warp

출시된지 몇 년 안된 신생 터미널 앱이지만 위에 소개한 iTerms2 과 거의 동일한 수준의 기본기능을 제공하고, 앱 내에서 제공하는 AI 기능의 편리함을 가진 터미널 앱. 게다가 Rust기반으로 작성되어 엄청나게 빠른 속도를 자랑하기 때문에 강력 추천!

[<img loading="lazy" class="aligncenter size-large wp-image-1103" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal-1024x698.png" alt="war" width="625" height="426" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal-1024x698.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal-300x204.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal-768x523.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal-1536x1047.png 1536w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal-2048x1395.png 2048w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal-150x102.png 150w" sizes="(max-width: 625px) 100vw, 625px" />][3]

  * <https://app.warp.dev/referral/X98J3>
  * 개인 사용은 무료로 가능
  * iTerms2가 가진 기본 기능들이 동일하게 구현되어있음
  * 자주 사용되는 명령어 도구들에 대해서 옵션 자동 완성 및 상세한 설명을 제공 
      * `ls -l` 을 입력했을때 나타나는 스크린샷 참고
  * AI 기능과 통합이 잘되어있음 
      * 터미널 창에서 바로 자연어로 물어봐서 적절한 명령어를 추천 받을 수 있음
      * 간단한 쉘 스크립트를 바로 AI에게 물어봐서 작성 가능
  * 자신의 터미널 설정들이 자동으로 Warp 계정에 연동되서, 여러 컴퓨터에서 동일한 사용자 경험을 얻을 수 있음

## Etc.

### Sequel Pro

DB로 MySQL을 사용한다면 무료 GUI MySQL client로는 Sequel Pro를 사용해볼만하다. [MySQL Workbench][4]가 기능이 훨씬 많지만 Mac 스럽지 않은 UI와 무거운 느낌때문에 비추천. 유료버전도 괜찮다면 아래 TablePlus를 더 적극 추천함!

<img loading="lazy" class="aligncenter size-large wp-image-847" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/sequel_pro_mac_app-1024x788.png" alt="" width="625" height="481" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sequel_pro_mac_app-1024x788.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sequel_pro_mac_app-300x231.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sequel_pro_mac_app-768x591.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sequel_pro_mac_app-150x115.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sequel_pro_mac_app.png 1902w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://www.sequelpro.com/>
  * MySQL에서 꼭 필요한 기능만 모아놓아서 적응하기 쉽고 UI가 심플하고 이쁘다.
  * ssh tunneling 지원
  * 무료 앱이지만 Donation을 받고있음 (저는 벌써 3년 전 쯤 기부했습니다!)
  * 최근 버전에서 랜덤한 크래시가 좀 잦아져서 약간 불편함

### TablePlus

이것도 또다른 GUI Database Client인데 필자는 지난 몇년간 더이상 업데이트가 이뤄지지 않는 Sequel Pro대신 TablePlus로 갈아탔다. (MySQL은 당연히 지원되고 그외에 지원되는 데이터베이스들도 엄청 많음)

[<img loading="lazy" class="aligncenter size-large wp-image-972" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus-1024x697.png" alt="" width="625" height="425" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus-1024x697.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus-300x204.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus-768x522.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus-1536x1045.png 1536w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus-2048x1393.png 2048w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus-150x102.png 150w" sizes="(max-width: 625px) 100vw, 625px" />][5]

  * <https://tableplus.com/>
  * 무료로 사용가능하나 탭을 2개까지밖에 열지 못해서 불편함
  * 유료는 $79 이지만 개발 생산성이 그만큼 올라가기 때문에 아낌없이 투자 가능.
  * DB 연결 정보마다 컬러링을 설정할 수 있어서 운영DB에 빨간색을 칠해두면 실수를 줄일 수 있음
  * 데이터 변경 내용을 모아서 리뷰한 후에 해당 쿼리들을 한번에 실행하는것이 가능 (실수 예방!)
  * 기존 작성된 쿼리를 저장하고, 폴더별로 그룹화 해서 관리할 수 있는 기능이 매우 편리 
      * 쿼리 작성/실행을 한 후 이슈 번호 또는 토픽 별로 그룹화 해서 저장해두면 나중에 셀프 참조해서 새로운 쿼리를 작성할 때 매우 유용
  * 단축키가 직관적이고 편리함 
      * CMD + n: 새로운 데이터베이스 연결
      * CMD + p: Open anything -> 이 단축키는 필수
      * CMD + k: Open database
      * CMD + t: 쿼리 작성창 열기
      * 데이터를 수정한 후 
          * CMD + SHIFT + p: 수정 내용 프리뷰
          * CMD + s: 수정 내용 커밋
  * 지원 데이터베이스 목록 
      * PostgreSQL, MySQL, SQLite, Microsoft SQL Server, Amazon Redshift, Oracle (Only macOS), CockroachDB, Snowflake (macOS and Windows), Cassandra, Redis, Vertica, MongoDB (Beta)

### Medis

Redis를 사용한다면 가끔 직접 Redis 서버에 접속해서 이것 저것 커맨드를 날리거나 내용을 읽고/수정할 일들이 많이 생긴다. 이때 GUI Redis client인 Medis를 사용해 보자.

<img loading="lazy" class="aligncenter size-large wp-image-846" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/medis_mac_app-1024x772.png" alt="" width="625" height="471" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/medis_mac_app-1024x772.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/medis_mac_app-300x226.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/medis_mac_app-768x579.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/medis_mac_app-150x113.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/medis_mac_app.png 1904w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <http://getmedis.com/>
  * 무료 앱이다!
  * ssh tunneling 지원
  * UI + CLI 동시 지원

### Xee3

빠르고 가벼운 맥용 이미지 뷰어를 찾는다면 Xee3를 추천한다. 안타깝게도 유료이지만 그렇게 비싼편도 아니고 subscription도 아니다. 개발자가 마음바뀌기 전에 어서 질러두자.

<img loading="lazy" class="aligncenter size-large wp-image-839" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/xee3_mac_app-1024x781.png" alt="" width="625" height="477" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/xee3_mac_app-1024x781.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/xee3_mac_app-300x229.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/xee3_mac_app-768x586.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/xee3_mac_app-150x114.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/xee3_mac_app.png 1616w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://theunarchiver.com/xee>
  * 유료 앱이다. $3.99
  * 맥의 기본앱인 Preview보다 빠르고 가벼운 이미지 뷰어
  * 스크롤 휠로 빠르게 여러 이미지를 브라우징 할 수 있다. (Xee에서 다음 이미지를 미리 로딩해서 캐시하는것으로 추측된다.)

### NameChanger

여러 파일들의 이름을 규칙에 맞게 일괄 변경을 하고 싶다면 이 앱을 강력 추천한다. 개발자라면 물론 터미널에서 모든걸 할 수 있지만 가끔은 GUI로 보면서 직관적으로 작업하고 싶을때가 있으니 부끄러워 하지 말자.

<img loading="lazy" class="aligncenter size-large wp-image-838" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/nameChanger_mac_app-1024x707.png" alt="" width="625" height="432" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/nameChanger_mac_app-1024x707.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/nameChanger_mac_app-300x207.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/nameChanger_mac_app-768x530.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/nameChanger_mac_app-150x104.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/nameChanger_mac_app.png 1484w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://www.mrrsoftware.com/namechanger/>
  * 무료 앱이다!
  * GUI기반의 파일명 일괄 변경
  * replace, prefix, postfix, regex 등등 다양한 조건 지원

### Sloth

실행중인 모든 프로세스들이 어떤 파일/디렉토리/소켓/파이프/디바이스를 열고 있는지를 GUI로 보여준다. 예를들어 서버가 정상 종료되지 않은 경우 가끔 포트를 계속 점유하는 케이스가 발생하는데 이때 터미널에서 `lsof -n -i:9110` 같은 명령어로 9110번 포트 어떤 프로세스가 쓰고있는지 pid 확인해서 kill 하는대신, UI에서 바로 검색하고 kill 할 수 있어서 편리하다.

  * <https://github.com/sveinbjornt/Sloth>
  * 무료 앱이다!

[<img loading="lazy" class="aligncenter size-large wp-image-1045" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill-1024x768.png" alt="" width="625" height="469" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill-1024x768.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill-300x225.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill-768x576.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill-1536x1151.png 1536w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill-2048x1535.png 2048w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill-150x112.png 150w" sizes="(max-width: 625px) 100vw, 625px" />][6]

### Hex Fiend

맥용 헥스 에디터(Hex Editor)를 찾고있다면 이녀석이 정답이다.

<img loading="lazy" class="aligncenter size-large wp-image-837" src="http://www.letmecompile.com/wp/wp-content/uploads/2019/02/hexfiend-1024x874.png" alt="" width="625" height="533" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2019/02/hexfiend-1024x874.png 1024w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/hexfiend-300x256.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/hexfiend-768x656.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/hexfiend-150x128.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2019/02/hexfiend.png 1354w" sizes="(max-width: 625px) 100vw, 625px" /> 

  * <https://ridiculousfish.com/hexfiend/>
  * 무료 앱이다!
  * 바이너리 파일을의 내용을 직접 보면서 편집 할 때 유용하다.
  * Binary diff 기능도 있어서 두 파일의 차이점도 빠르게 비교할 수 있다.

 [1]: http://www.letmecompile.com/wp/wp-content/uploads/2019/02/proxyman.png
 [2]: https://www.charlesproxy.com/
 [3]: http://www.letmecompile.com/wp/wp-content/uploads/2019/02/warp-terminal.png
 [4]: https://www.mysql.com/products/workbench/
 [5]: http://www.letmecompile.com/wp/wp-content/uploads/2019/02/tableplus.png
 [6]: http://www.letmecompile.com/wp/wp-content/uploads/2019/02/sloth_find_port_and_kill.png