---
title: 크롬 익스텐션 개발 + 리액트 적용하기
date: 2018-07-21T17:05:44+00:00
url: /chrome-extension-with-react/
dsq_thread_id:
  - 6808756131
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/steemit-font-changer/"     class="post-717"><span class="crp_title">스팀잇 폰트체인저 - 한글 폰트 최적화로 스팀잇 포스트의 가독성을 향상시키기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/steemit-font-changer/"     class="post-717"><span class="crp_title">스팀잇 폰트체인저 - 한글 폰트 최적화로 스팀잇 포스트의 가독성을 향상시키기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
tags:
  - javascript
  - chrome
  - react
  - boilerplate
  - 리액트
  - redux
  - 템플릿

---
크롬 익스텐션(Chrome Extension)은 모두 자바스크립트(Javascript)로 되어있기때문에 웹개발을 해본 사람이라면 그리 큰 노력을 들이지 않고도 개발을 시작 할 수 있다. 하지만 구글에서 익스텐션을 통한 보안이슈가 생기는 것을 막기 위해 다양한 방식으로 익스텐션의 자바스크립트가 실행되는 컨텍스트(context)들을 세분화 시켜두었기 때문에, 이러한 세분화된 컨텍스트에서 각각 어떤 작업이 수행가능한지, 어떻게 분리된 스크립트 컨텍스트간에 통신을 할 수 있는지를 이해하는 것이 생각보다 어렵다.

따라서 본 글에서는 이 부분의 개념에 대해서 자세히 비교 설명하고, 각각의 스크립트들이 잘 동작하도록 미리 설정된 보일러 플레이트 프로젝트를 소개할 예정이다. 대신 크롬 익스텐션에서 사용하는 세부적인 API들이나 manifest관련 세부 설정들은 구글에서 제공하는 문서에도 잘 나와있고, 다양한 sample project에도 케이스별로 잘 나와있기 때문에 따로 정리하지는 않을 예정이다.

## 스크립트의 컨텍스트

크롬 익스텐션과 그와 관련된 스크립트가 실행되는 컨텍스트에 따라서 page, background, content, injected script 4가지로 구분을 할 수 있다. 각각의 컨텍스트가 어떻게 다른지 좀 더 자세히 살펴보도록 하자.

### 1. Page script

  * 크롬 익스텐션 문서에서 사용하는 정식 명칭은 아니지만 아래 설명할 다른 script들의 구분을 위해 현재 열려있는 페이지에서 실행되고 있는 일반적인 스크립트를 page script라고 지칭한다 (크롬 익스텐션과 무관)
  * page script는 현재 open된 페이지 자체의 script이고, 크롬 익스텐션에 포함되는 script가 아니다.
  * 크롬 익스텐션에 포함된 script를 이용해서 page script와 상호작용 (page script 내부의 변수를 액세스하거나, 메서드를 호출하는 등) 을 하려면 익스텐션에 포함된 script를 page context로 inject해서 사용해야 한다 (아래쪽에 설명된 injected script 참고)

### 2. Background script

  * <https://developer.chrome.com/extensions/background_pages>
  * 크롬 어플리케이션의 background에서 실행되는 스크립트 
  * 페이지이동, 탭 열기/닫기, 북마크 추가/삭제 등 의 이벤트를 모니터링 할 수 있음 
      * `chrome.*` 모든 API에 접근 가능
      * 크롬에서 제공하는 어플리케이션 레벨 API에 접근 할 수 있는 권한이 가장 막강하지만, 반대로 유저가 보고있는 페이지 레벨에 대해서는 직접적으로 접근이 제한되어있다. 때문에 다른 영역의 script들과 통신하기 위해서는 Message passing 방식을 사용하여 간접적으로 접근하는 것만 가능하다.
  * 메시지 패싱 방식을 사용시 모든 메시지 포트가 닫힐 때 까지는 unload되지 않는다.
  * 디버깅 방법 
      * <chrome://extensions> 를 열어서 load된 개발중인 익스텐션을 보면 &#8220;Inspect views background.html&#8221; 이라는 메뉴가 있다.
      * persistent: false 로 manifest에 설정된 경우 idle 상태로 빠지게되면 더이상 디버깅이 불가능해지므로 개발중에는 persistent: true로 설정 해 두는 것이 편리하다.

### 3. Content script

  * <https://developer.chrome.com/extensions/content_scripts>
  * 현재 사용자가 보고있는 페이지의 컨텍스트에서 실행되는 스크립트 
      * 현재 페이지의 DOM을 읽어와서 조작이 가능하다.
      * 하지만 현재 페이지의 DOM만을 공유할 뿐 실제 page script와는 완전히 isolated된 상태로 실행된다. 
          * 즉, page script에서 로드된 된 라이브러리, 변수, 함수 등에 접근하는것이 불가능하다.
      * `chrome.runtime.*` API에 접근 가능
  * 앞서 설명했듯이, 다른 컨텍스트의 script들과는 직접적인 접근이 막혀있기때문에 메시지 패싱 방식을 통해 통신을 해야한다. 케이스별로 조금씩 API가 다르지만 원리는 동일하다. 
      * background script와 정보 교환 
          * `chrome.runtime.onMessage`
          * `chrome.runtime.sendMessage`
      * page script와 정보 교환 
          * `window.postMessage`
          * `var port = chrome.runtime.connect(); port.postMessage`
  * 익스텐션 내부의 파일을 로드할 경우 다음 API 를 통해서 액세스 가능 
      * `chrome.runtime.getURL(path)`
  * Content Scripts 를 로드하는 두가지 방식 
      * load content script programmatically from background script 
          * activeTab 권한 추가 필요 
              * 이 권한이 설정되면 content script가 current active tab에 로드되는 것이 가능
          * 다음 코드를 실행하면 코드 블록 또는 코드 파일이 content script로써 로드된다. 
              * `chrome.tabs.executeScript({ code: '...'})`
              * `chrome.tabs.executeScript({ file: 'file.js'})`
      * declaratively 
          * manifest파일의 &#8216;content_scripts&#8217; 섹션에 추가 
                "content_scripts": [
                    {
                     "matches": ["http://*.nytimes.com/*"],
                     "css": ["myStyles.css"],
                     "js": ["contentScript.js"]
                    }
                ],
                

  * 디버깅 방법 
      * content script가 실행되도록 설정된 페이지가 로드되면 inspector를 열어서 Sources &#8211; Content scripts 영역에서 스크립트를 확인 할 수 있다.

### 4. Injected script

  * page script의 컨텍스트에서 실행되는 코드가 필요하다면, content script의 컨텍스트로부터 script를 inject해야한다.
  * inject된 script의 경우 page script 컨텍스트에서 실행되므로 `chrome.*` 의 모든 API에대해서는 접근 불가
  * 방법 1: 파일 로드 
         var s = document.createElement('script');
        // TODO: add "script.js" to web_accessible_resources in manifest.json
        s.src = chrome.extension.getURL('script.js');
        s.onload = function() {
            this.remove();
        };
        (document.head || document.documentElement).appendChild(s);
        
    
      * manifest에 `web_accessible_resources` 항목을 아래와 같이 추가하면, 익스텐션 패키지에 포함된 파일을 로드하는 것이 가능해진다. [구글 문서 참고][1] 
               web_accessible_resources": ["script.js"]
            
    
      * CSP (Content Security Policy)가 설정된 웹사이트에서는 미리 설정된 domain에 대해서만 remote script를 load가능하기 때문에 함부로 아무 스크립트나 로드하는것이 불가능 하므로, inject할 스크립트는 되도록 익스텐션 패키지 안에 포함시키는 것이 안전하다.</p> 

  * 방법2: 인라인(inline) 방식
    ```javascript
    var actualCode = '(' + function() {
        // All code is executed in a local scope.
        // For example, the following does NOT overwrite the global `alert` method
        var alert = null;
        // To overwrite a global variable, prefix `window`:
        window.alert = null;
    } + ')();';
    var script = document.createElement('script');
    script.textContent = actualCode;
    (document.head||document.documentElement).appendChild(script);
    script.remove();
    ```

  * 참고: <https://stackoverflow.com/questions/9515704/insert-code-into-the-page-context-using-a-content-script/9517879#9517879> 

### 정리

앞서 살펴본 각각의 컨텍스트를 염두에 두고, 익스텐션을 개발할 때 기능에따라 어떤 컨텍스트에서 코드를 작성할지 정하고 시작하면 좀 덜 헷갈린다. 예를들어 탭관련 상태정보를 얻고싶다면 Chrome Application과 과련된 API를 사용해야 하기때문에 background script를 작성해야하고, 현재보고있는 페이지에 새로운 버튼을 추가하고 싶다면 content script를 작성해야하는 식이다.

## React 로 Chrome extension 개발하기

Plain Javascript만으로도 크롬 익스텐션을 개발하는데 아무 문제가 없지만, React나 Vuejs 등의 프레임워크를 사용하면 UI가 있는 Chrome Extension을 만드는데 한결 생산성이 높아진다.

  * 보일러플레이트 (boilerplate) 
      * 크롬 익스텐션과 리액트가 합쳐진 개발환경을 구성하려면 설정에 손이 많이가는데, 보일러 플레이트 코드를 사용하면 별다른 설정 없이도 쉽게 실제 개발을 시작 할 수 있다.
      * <https://github.com/jhen0409/react-chrome-extension-boilerplate> 
          * Webpack3, React 15로 구성되어있고 page context에 script를 inject 하는것이 불가능하다. (injected script 기능이 없음)
      * <https://github.com/yjiq150/react-chrome-extension-boilerplate> 
          * 위의 보일러플레이트 프로젝트를 기반으로 Webpack 4, React 16으로 업데이트하고, page context script inject를 지원하도록 개발환경을 변경해 둔 버전이다. (원 저작자에게 해당 수정사항에 대한 Pull Request를 올려둔 상태)
          * 스크립트 구성 설명 
              * window: 새창을 띄워서 원하는 페이지/스크립트를 로딩한다
              * popup: 크롬 상단바의 익스텐션 아이콘이 나오는 영역에서 해당 익스텐션을 클릭했을때 나오는 팝업 페이지
              * background: 익스텐션에 저장되는 데이터 관리 및 페이지, 탭의 상태변화 감지 등의 역할 수행
              * content: 현재 사용자가 보고있는 로드된 페이지의 DOM영역에 접근하거나, 새로운 css style을 로드 가능
              * page (injected script): 현재 사용자가 보고있는 로드된 페이지에 extension 에 미리 포함되어 패키징된 script를 inject해서 실행 가능
  * 실제 적용 샘플 
      * 위의 보일러 플레이트를 이용해서 직접 만든 [스팀잇 인핸서][2] 크롬 익스텐션을 소개한다.
      * 소스코드도 모두 공개되어있다. <https://github.com/yjiq150/steemit-enhancer>
  * 참고: [Extension Reloader][3] 익스텐션 
      * 보일러 플레이트를 사용하면 HMR (hot module replacement) 또는 “webpack watch 모드 + 수동 페이지 새로고침” 을 통해서 변경된 코드를 반영하는 것이 가능하다. 하지만 크롬 익스텐션의 설정자체가 변경되거나 스크립트 파일이 추가/삭제 되는 등의 상황이 발생하면 결국 크롬 익스텐션 자체를 다시 로드해줘야 한다.
      * extension reloader 익스텐션을 사용하면 원클릭으로 좀더 쉽게 익스텐션 릴로드가 가능하다. 
          * <http://reload.extensions>를 열면 자동으로 unpacked extension들이 reload된다.

 [1]: https://developer.chrome.com/extensions/manifest.html#web_accessible_resources
 [2]: https://chrome.google.com/webstore/detail/steemitcom-enhancer/gepddagaajclednflogimfmpoejjkcnj
 [3]: https://chrome.google.com/webstore/detail/extensions-reloader/fimgfedafeadlieiabdeeaodndnlbhid
