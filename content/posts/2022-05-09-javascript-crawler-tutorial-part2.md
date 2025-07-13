---
title: '자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기'
date: 2022-05-09T14:33:02+00:00
url: /javascript-crawler-tutorial-part2/
cover:
  image: "/uploads/2022/05/javascript-crawler-tutorial-og-02.png"
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-intro/"     class="post-1007"><span class="crp_title">자바스크립트로 크롤러 만들기: 크롤링 개념 및 튜토리얼 소개</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-intro/"     class="post-1007"><span class="crp_title">자바스크립트로 크롤러 만들기: 크롤링 개념 및 튜토리얼 소개</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - Javascript

---
웹 크롤링을 하려면 웹페이지 구조인 DOM과 CSS 셀렉터 문법을 알아야 합니다. 각각을 알아봅시다.

## 웹페이지와 DOM {#toc_1}

웹페이지는 HTML 형식으로 제공되는 일종의 문서라고 생각할 수 있습니다. 웹 브라우저로 웹페이지에 접근한다는 것은, 간단히 말해 서버로부터 해당 주소에서 제공하는 HTML 문서를 HTTP 통신으로 전달받는 것을 의미합니다. 전달받은 HTML 문서는 단순 텍스트 형태이기 때문에 프로그램에서 사용하기 좋은 데이터 구조로 표현해야 하는데, 이 구조를 DOM이라고 부릅니다. DOM<sup>Document Object Model</sup>은 최상위 노드<sup>node</sup>와 여러 단계의 자식 노드들로 구성된 트리<sup>tree</sup> 구조입니다. 따라서 원하는 노드를 쉽게 찾아서 수정/삭제하거나 원하는 위치에 새로운 노드를 추가할 수 있습니다.

<img loading="lazy" width="632" height="702" src="/uploads/2022/05/4-11_dom.png" alt="[그림 4-11] DOM 트리와 요소" class="wp-image-989" srcset="/uploads/2022/05/4-11_dom.png 632w, /uploads/2022/05/4-11_dom-270x300.png 270w, /uploads/2022/05/4-11_dom-150x167.png 150w" sizes="(max-width: 632px) 100vw, 632px" /> 

요소<sup>element</sup>는 DOM 트리 구조안에 존재하는 다양한 노드 중 element라는 타입의 노드를 의미합니다. [그림 4-11]에서<sup id="fnref1"><a href="#fn1" rel="footnote">1</a></sup> 볼 수 있듯이 보통 하나의 HTML 태그가 하나의 요소로 표현됩니다. 반면 태그 사이의 텍스트 값은 text 타입의 노드이며, href 태그에 지정된 속성<sup>attribute</sup>은 attribute 타입의 노드입니다.

웹 브라우저는 HTML을 DOM으로 변환한 후, CSS로 설정한 스타일을 DOM의 각 노드에 적용하여 사용자에게 보여줍니다. 이것이 우리가 보는 웹페이지 화면입니다. 사용자의 인터랙션에 따라 페이지 일부를 변경하고 싶다면 자바스크립트를 이용해 DOM의 일부를 원하는 형태로 변경하면 됩니다. DOM이 바뀌면 해당 부분의 화면이 다시 렌더링되고, 사용자가 보는 화면도 바뀌게 됩니다.

이제 웹페이지에서 HTML과 DOM의 기본적인 개념은 이해했습니다. 웹페이지를 크롤링하려면 DOM을 이용하여 HTML 문서에서 원하는 요소를 찾아내야 합니다. 요소를 찾는 방법을 알아보겠습니다.

#### HTML 문서에서 원하는 DOM 요소 찾기 {#toc_2}

예제 페이지 <https://yjiq150.github.io/coronaboard-crawling-sample/dom>를 크롬 웹 브라우저에서 열고 [개발자 도구]로 해당 페이지를 살펴보면서 설명을 진행하겠습니다.

위 웹페이지가 로드된 상태로 크롬 [개발자 도구]의 [Element] 탭을 살펴보면 현재 페이지를 구성하는 HTML이 나타납니다. HTML은 결국 텍스트 편집기로도 쉽게 편집하는 다음과 같이 구조화된 텍스트 문서임을 알 수 있습니다.

<img loading="lazy" width="1012" height="616" src="/uploads/2022/05/4_dom_example.png" alt="" class="wp-image-990" srcset="/uploads/2022/05/4_dom_example.png 1012w, /uploads/2022/05/4_dom_example-300x183.png 300w, /uploads/2022/05/4_dom_example-768x467.png 768w, /uploads/2022/05/4_dom_example-150x91.png 150w" sizes="(max-width: 1012px) 100vw, 1012px" /> 

프로그래밍 관점에서 구조가 없는 문서에서 특정 위치에 접근해 내용만 추출하거나, 삭제하는 일은 쉽지 않습니다. 하지만 구조화된 텍스트 문서에서는 매우 쉽게 할 수 있습니다. 예를 들어 제목을 추출하는 방법을 생각해봅시다. 먼저 `html` 태그를 찾고 ➝ `html` 태그 안에 존재하는 `body` 태그를 찾고 ➝ `body` 태그 안에 존재하는 `h1` 태그를 찾으면 됩니다.

구조화된 텍스트 문서를 프로그래밍을 통해 접근하여 내용이나 스타일, 구조 등을 수정하기 쉽도록 모델링해서 사용하는 방식이 앞서 언급한 DOM입니다. 대부분의 웹 브라우저는 DOM에 접근하는 인터페이스를 표준에 따라 구현합니다. 그래서 웹 브라우저가 읽은 HTML 문서를 직접 파싱할 필요 없이 자바스크립트 코드 몇 줄로 손쉽게 DOM에 접근할 수 있습니다.

다음 예제코드를 한번 살펴봅시다.

```javascript
const elements = document.getElementsByTagName('h1');
console.log(elements[0].textContent); // 코로나보드
```

크롬 ❶ [개발자 도구]의 콘솔창에서 ❷ 코드를 실행하면 ❸ ‘코로나보드’가 출력됩니다.

<img loading="lazy" width="892" height="278" src="/uploads/2022/05/4_dom_console_run.png" alt="" class="wp-image-991" srcset="/uploads/2022/05/4_dom_console_run.png 892w, /uploads/2022/05/4_dom_console_run-300x93.png 300w, /uploads/2022/05/4_dom_console_run-768x239.png 768w, /uploads/2022/05/4_dom_console_run-150x47.png 150w" sizes="(max-width: 892px) 100vw, 892px" /> 

HTML 태그들은 DOM 트리 안에서 요소<sup>element</sup>로 표현되어 모두 부모<sup>parent</sup>/자식<sup>child</sup> 관계로 연결됩니다. 이러한 구조화된 트리 데이터 구조가 있기 때문에 트리의 최상위 객체를 기준으로 원하는 요소를 찾을 수 있습니다. 웹 브라우저에서 `document` 객체는 현재 로드된 웹페이지를 나타내고, 현재 보고 있는 웹페이지의 DOM 트리<sup>tree</sup>의 최상위 요소입니다.

요소 객체에는 DOM을 이용하여 해당 요소 서브의 요소들을 찾을 수 있게 해주는 다양한 함수들이 구현되어 있습니다. 위 코드에서는 요소 객체에 제공하는 함수 중 하나인 `getElementsByTagName()`을 호출하여 웹페이지에 존재하는 `h1` 태그로 정의된 요소들을 모두 찾은 후, 그중 첫 번째 요소의 `textContent` 속성에 접근하여 ‘코로나보드’라는 텍스트를 추출합니다.

▽ 찾기 함수

<table>
  <tr>
    <td>함수</td>
    <td>설명</td>
  </tr>
  <tr>
    <td>
      <strong>getElementsByTagName()</strong>
    </td>
    <td>태그 이름으로 찾기</td>
  </tr>
  <tr>
    <td>
      <strong>getElementsByClassName()</strong>
    </td>
    <td>클래스 속성값으로 찾기</td>
  </tr>
  <tr>
    <td>
      <strong>getElementById()</strong>
    </td>
    <td>문서 전체에 대하여 유니크한 id값을 가진 요소를 찾기. document 객체에서만 호출이 가능합니다.</td>
  </tr>
</table>

#### 요소 찾는 방법 비교 getElement* vs. querySelector {#toc_3}

앞에서 소개한 함수들은 간단한 조건만으로 요소를 찾는 데 유용합니다. 하지만 복잡한 조건으로 검색하는 것은 쉽지 않습니다. 예를 들어 위 예제의 HTML에서 ‘slide’라는 클래스 속성값을 가진 요소의 서브에 있는 `p` 태그의 내용만 출력할 때는 다음과 같이 두 단계로 코드를 작성해야 합니다.

```javascript
// 'slide' 클래스 속성값을 가진 요소를 먼저 찾음
const slideElements = document.getElementsByClassName('slide');

for (const slideElement of slideElements) {
  // 찾은 요소를 기준으로 다시 p 태그 검색
  const paragraphElements = slideElement.getElementsByTagName('p');
  for (const paragraphElement of paragraphElements) {
     // 검색한 p 태그의 내용 출력
     console.log(paragraphElement.textContent);
  }
}
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">국가별 내용
대한민국 내용</code></pre>
</div>

두 단계의 조건만으로도 코드가 복잡합니다. 실제 웹페이지는 더 복잡하므로 더 간단한 해결책이 필요합니다. 이때 CSS 셀렉터를 사용하면 간단히 해결을 할 수 있습니다. 요소에서 제공하는 `querySelectorAll()` 함수(조건을 만족하는 모든 항목 찾기) 또는 `querySelector()` 함수(조건을 만족하는 첫 번째 항목 찾기)를 사용하면 CSS에서 사용하는 셀렉터 문법을 통해서 손쉽게 조건에 맞는 요소를 찾을 수 있습니다.

위 예제와 같은 조건을 CSS 셀렉터를 이용하여 다시 작성하면 다음과 같습니다.

```javascript
const paragraphElements = document.querySelectorAll('.slide p');
for (const paragraphElement of paragraphElements) {
  console.log(paragraphElement.textContent);
}
```

`querySelectorAll()` 함수와 `.slide p`라는 CSS 셀렉터를 이용하여 한 번에 원하는 요소를 찾아냈습니다. 웹페이지 내에서 원하는 조건에 맞는 요소를 빠르게 찾아내는 이러한 편리함 덕분에 실제 웹페이지 크롤링에서는 대개 CSS 셀렉터 문법을 사용합니다. 자주 사용하는 셀렉터 문법을 알아보겠습니다.

## CSS 셀렉터 문법 {#toc_4}

기본 셀렉터 사용법을 알아보고 나서 기본 셀렉터를 조합해 복잡한 조건을 적용하는 방법을 알아봅시다.

#### 기본 셀렉터 {#toc_5}

기본적인 CSS 셀렉터를 이용하면 명시한 태그, id, 클래스, 속성을 만족하는 요소들을 찾는 기능을 제공합니다. 기본 셀렉터 기능은 앞서 살펴보았던 `getElementsByTagName()`, `getElementById()`, `getElementByClassName()` 함수와 비슷합니다만, 조합하거나 계층 구조 조건을 적용하면 훨씬 정확한 검색을 할 수 있습니다. 복잡한 사용법을 익히기 전에 기본 사용 방식부터 살펴봅시다.

참고로 document 객체는 DOM의 최상위 요소를 의미합니다. 여기서 사용하는 예제들은 모두 `document`에 대해 `querySelectorAll()` 함수를 실행하여 웹페이지 전체를 검색합니다. 하지만 원한다면 `document`뿐만 아니라 다른 요소를 대상으로 `querySelectorAll()` 함수를 실행할 수 있습니다. 하면 해당 요소가 가진 자식 요소만 검색하므로 검색 범위가 좁아져 더 나은 성능을 얻을 수 있습니다.

##### 태그 셀렉터<sup id="fnref2"><a href="#fn2" rel="footnote">2</a></sup> {#toc_6}

태그 셀렉터<sup>type selector</sup>는 태그 이름을 그대로 적어주면 해당 태그에 해당하는 요소를 검색합니다.

[예시1]

```javascript
// h1 태그에 해당하는 요소를 검색
document.querySelectorAll('h1');
// 결과:
// <h1>코로나보드</h1>

// p 태그에 해당하는 요소를 검색
document.querySelectorAll('p');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;p&gt;코로나보드를 소개합니다&lt;/p&gt;
&lt;p&gt;국가별 내용&lt;/p&gt;
&lt;p&gt;대한민국 내용&lt;/p&gt;</code></pre>
</div>

##### ID 셀렉터 {#toc_7}

ID 셀렉터<sup>ID selector</sup>는 찾고자 하는 `id` 속성값 앞에 `#` 기호를 붙여주어 검색합니다(HTML 문서 내에서 id값은 항상 유니크하기 때문에 하나만 존재합니다).

[예시1]

```javascript
// id="country-title"인 요소를 검색
document.querySelectorAll('#country-title');
```

[찾아지는 엘리먼트]

<div>
  <pre><code>&lt;h2 id="country-title"&gt;국가별 현황&lt;/h2&gt;</code></pre>
</div>

##### 클래스 셀렉터 {#toc_8}

클래스 셀렉터<sup>class selector</sup>는 찾고자 하는 클래스 속성값의 앞에 `.` 기호를 붙여 검색합니다.

[예시1]

```javascript
// class="slide" 인 요소를 검색
document.querySelectorAll('.slide');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;div class="slide"&gt;...&lt;/div&gt;
&lt;div class="slide"&gt;...&lt;/div&gt;
&lt;div class="slide"&gt;...&lt;/div&gt;</code></pre>
</div>

##### 속성 셀렉터 {#toc_9}

속성 셀렉터<sup>attribute selector</sup>는 찾고자 하는 속성 이름을 `[ ]`로 묶어 검색합니다. 해당 속성의 값도 지정하여 검색할 수 있습니다.

[예시1]

```javascript
// "type"이라는 속성을 가진 요소 검색
document.querySelectorAll('[type]');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;input class="form-control" type="text" value="입력창"&gt;
&lt;input type="password" value="1234"&gt;</code></pre>
</div>

[예시2]

```javascript
// "type"이라는 속성의 값이 "text"인 요소 검색
document.querySelectorAll('[type="text"]');
```

[찾아지는 엘리먼트]

<div>
  <pre><code>&lt;input class="form-control" type="text" value="입력창"&gt;</code></pre>
</div>

#### 셀렉터 조합하기 {#toc_10}

요소가 많은 경우 기본 셀렉터만으로 찾기가 쉽지 않습니다. 기본 셀렉터들을 조합하여 사용하는 방식을 살펴봅시다.

##### OR 조건으로 찾기 {#toc_11}

여러 기본 셀렉터를 `,`로 연결하면 각 조건을 만족하는 요소 모두를 검색합니다.

[예시1]

```javascript
// h1 태그 또는 'slide' 클래스 속성값을 가진 요소를 검색
document.querySelectorAll('h1, .slide');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;h1&gt;코로나보드&lt;/h1&gt;
&lt;div class="slide"&gt;...&lt;/div&gt;
&lt;div class="slide"&gt;...&lt;/div&gt;
&lt;div class="slide"&gt;...&lt;/div&gt;</code></pre>
</div>

##### AND 조건으로 찾기 {#toc_12}

다른 종류의 기본 셀렉터를 띄어쓰기 없이 이어 붙이면 해당 조건들을 동시에 만족하는 요소를 검색합니다. 이어붙일 때 태그 셀렉터가 제일 앞에 나와야 하고 클래스 셀렉터나 속성 셀렉터는 순서에 관계없이 태그 셀렉터 뒤쪽에 있도록 하면 됩니다.

[예시1]

```javascript
// div 태그에 'slide' 클래스 속성값이 지정된 요소 검색
document.querySelectorAll('div.slide');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;div class="slide"&gt;...&lt;/div&gt;
&lt;div class="slide"&gt;...&lt;/div&gt;
&lt;div class="slide"&gt;...&lt;/div&gt;</code></pre>
</div>

[예시2]

```javascript
// input 태그에 type 속성값이 'password'인 요소 검색 
document.querySelectorAll('input[type="password"]');
```

[찾아지는 엘리먼트]

<div>
  <pre><code>&lt;input type="password" value="1234"&gt;</code></pre>
</div>

[예시3]

```javascript
// input 태그에 'form-control' 클래스 속성값이 지정되어 있고
// type 속성값이 'text'인 요소 검색 
document.querySelectorAll('input.form-control[type="text"]');
```

[찾아지는 엘리먼트]

<div>
  <pre><code>&lt;input class="form-control" type="text" value="입력창"&gt;</code></pre>
</div>

#### 계층 구조 조합하기 {#toc_13}

앞서 설명했듯이 HTML은 트리 형태의 데이터 구조인 DOM으로 표현될 수 있습니다. DOM은 요소 간에 부모/자식의 계층 구조를 갖습니다. 이러한 계층 구조를 이용하여 원하는 요소를 CSS 셀렉터 문법으로 찾는 방법을 알아봅시다.

##### 계층 순서로 요소 찾기 {#toc_14}

여러 셀렉터를 공백으로 연결하면, 연결된 순서대로 부모/자식 계층 관계를 가지는 요소를 검색합니다. 이때 꼭 부모/자식 관계뿐 아니라 세대를 건너뛰는 요소도 검색됩니다.

[예시1]

```javascript
// div 태그를 상위 요소로 가진 모든 p 태그 요소를 검색
document.querySelectorAll('div p');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;p&gt;코로나보드를 소개합니다&lt;/p&gt;
&lt;p&gt;국가별 내용&lt;/p&gt;
&lt;p&gt;대한민국 내용&lt;/p&gt;</code></pre>
</div>

[예시2]

```javascript
// div 태그를 상위 요소로 두 번 가진 모든 p 태그 요소를 검색
document.querySelectorAll('div div p');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;p&gt;국가별 내용&lt;/p&gt;
&lt;p&gt;대한민국 내용&lt;/p&gt;</code></pre>
</div>

##### 직접적인 자식 요소 찾기 {#toc_15}

여러 기본 셀렉터를 > 기호를 사용하여 연결하면 직접적인 부모/자식 계층 관계를 가지는 요소를 검색합니다. 직접적인 계층 관계를 명시하면 계층 순서로 요소를 찾을 때보다 찾고자 하는 요소를 더 정확히 찾을 수 있습니다.

[예시1]

```javascript
// 'container' 클래스 속성값을 가진 div 요소의 직접적인 자식 중 p 태그 요소를 검색
document.querySelectorAll('div.container > p');
```

[찾아지는 엘리먼트]

<div>
  <pre><code>&lt;p&gt;코로나보드를 소개합니다&lt;/p&gt;</code></pre>
</div>

##### 동일한 부모를 가진 요소 찾기 {#toc_16}

여러 기본 셀렉터를 `~` 기호를 사용하여 연결하면 찾은 요소를 기준으로 동일한 부모를 갖는 조건을 만족하는 모든 요소를 검색합니다.

[예시1]

```javascript
// id="input-test-title"인 요소와 동일한 부모를 가진 input 태그 요소 검색
document.querySelectorAll('#input-test-title ~ input');
```

[찾아지는 엘리먼트]

<div>
  <pre><code class="language-none">&lt;input class="form-control" type="text" value="입력창"&gt;
&lt;input type="password" value="1234"&gt;</code></pre>
</div>

##### 동일한 부모를 가지면서 인접한 요소 찾기 {#toc_17}

여러 기본 셀렉터를 `+` 기호를 사용하여 연결하면 찾은 요소를 기준으로 동일한 부모를 가지면서 해당 요소 바로 다음에 나오는 요소 하나를 검색합니다.

```javascript
// id="input-test-title"인 요소와 동일한 부모를 가진 input 태그 요소를 검색
document.querySelectorAll('#input-test-title + input');
```

[찾아지는 엘리먼트]

<div>
  <pre><code>&lt;input class="form-control" type="text" value="입력창"&gt;</code></pre>
</div>

## 마무리 {#toc_18}

웹페이지가 DOM을 통해 어떻게 모델링이 되는지를 살펴봤고, 이 DOM에서 원하는 데이터의 위치를 파악하고 추출하기 위한 도구인 CSS 셀렉터를 사용하는 방식을 익혀보았습니다. 다음 3편에서는 다양한 유형의 웹페이지를 살펴보고 유형별로 어떻게 크롤러를 만드는 것이 가장 효율적인지 살펴보겠습니다.

##  {#toc_19}

  * [자바스크립트로 크롤러 만들기 소개: 크롤링 개념 및 튜토리얼 소개][1]
  * [자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기][2]
  * [자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기][3]
  * [자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기][4]
  * [자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기][5]

##  {#toc_20}

<div style="text-align: center;">
  <a class="coronaboard-book-banner mobile-only" href="https://bit.ly/3ymH6eK" target="_blank" rel="noopener noreferrer"><img src="https://coronaboard.kr/images/square-banner.png" alt="코로나보드로 배우는 실전 웹 서비스 개발" style="width: 320px; height:306px;" /></a>
</div>

[&#8216;코로나보드로 배우는 실전 웹 서비스 개발&#8217;][6] 책에서는 크롤러 뿐만 아니라 구글 스프레드 시트를 저장소로 사용하는 방법, 개츠비 + 리액트를 사용해서 웹사이트를 개발하는 방법, 도메인 설정, 검색엔진 최적화, 사용자 분석, 광고를 통한 수익화 등을 다룹니다. 웹서비스 개발과 운영에 대한 전반적인 내용을 알고싶으신 분들이나, 사이드 프로젝트를 만들어보고 싶으신 분들에게 추천합니다.

### 각주 {#toc_21}

<div class="footnotes">
  <ol>
    <li id="fn1">
      출처 &#8211; <a href="https://en.wikipedia.org/wiki/Document_Object_Model">https://en.wikipedia.org/wiki/Document<em>Object</em>Model</a> &nbsp;<a href="#fnref1" rev="footnote">↩</a>
    </li>
    <li id="fn2">
      모질라(Mozilla) 문서에는 타입 셀렉터(type selector)라고 정의되어 있습니다. 실제로는 태그를 선택하는 거라 태그 셀렉터가 더 명확한 것 같아, 이 책에서는 타입 셀렉터가 아닌 태그 셀렉터로 지칭했습니다.&nbsp;<a href="#fnref2" rev="footnote">↩</a>
    </li>
  </ol>
</div>

 [1]: https://www.letmecompile.com/javascript-crawler-tutorial-intro/
 [2]: https://www.letmecompile.com/javascript-crawler-tutorial-part1/
 [3]: https://www.letmecompile.com/javascript-crawler-tutorial-part2/
 [4]: https://www.letmecompile.com/javascript-crawler-tutorial-part3/
 [5]: https://www.letmecompile.com/javascript-crawler-tutorial-part4/
 [6]: https://bit.ly/3ymH6eK
