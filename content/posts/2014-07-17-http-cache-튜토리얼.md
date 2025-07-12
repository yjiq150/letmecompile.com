---
title: HTTP Cache 튜토리얼
author: yjiq150
type: post
date: 2014-07-17T00:25:07+00:00
url: /http-cache-튜토리얼/
dsq_thread_id:
  - 2849728273
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 프로그래밍 기타
  - 웹 개발
tags:
  - http cache tutorial
  - 캐쉬 설명
  - 캐쉬 지시자
  - cache control
  - cache directive

---
# HTTP Cache 튜토리얼

HTTP를 이용하는 어플리케이션을 개발하다보면 효율적인 네트워크 송수신을위해 서버/클라이언트에서 캐쉬(cache)를 이용하는것이 필수적이다. HTTP를 이용할때 어떤식의 캐쉬방식이 있는지, 어떤 종류의 캐쉬들이 있는지, 어플리케이션 개발에있어서 상식적으로 알고있어야 할 내용들을 정리해보았다.

## 캐쉬의 종류

캐쉬의 위치에 따라 다음과 같이 분류가 가능하다.<sup id="fnref-405-cache"><a href="#fn-405-cache" rel="footnote">1</a></sup>

  * Browser cache 
      * 웹브라우져 혹은 HTTP요청을 하는 클라이언트 어플리케이션들이 내부적으로 갖고있는 캐시이다.
  * Proxy cache 
      * 실제 서버가 있는곳이 아닌 네트워크 관리자에의해 네트워크상에 설치되는하는 캐시다.
      * 일반적으로 큰회사나 ISP의 방화벽(firewall)에 설치된다.
      * shared cache의 일종으로 많은 수의 사용자들에 의해 공유되어 사용되며, 레이턴시와 트래픽을 줄이는데 매우 도움이된다.
  * Gateway cache(reverse proxy cache) 
      * 네트워크상에 설치되지 않고 실제 서버의 관리자에의해 설치 및 운영된다.
      * 실제 서버의 앞단에 설치되어 요청에대한 캐쉬 및 효율적인 분배를 통해 서버의 응답 성능을 좋게하고, scalable하게 만들어 준다. 
      * 로드밸런서 등을 사용해서 실제 서버가 아닌 gateway cache로 요청을 reroute한다.
      * CDN은 이런 gateway 캐시를 유료로 제공해주는 서비스라고 볼 수 있다.

## 기본적인 캐쉬 동작 방식

  * 응답 헤더의 캐쉬가 캐쉬 하지말라고 지정되어있는 경우 캐쉬하지 않는다</p> 
  * 요청이 HTTPS거나 HTTP 인증을 사용한 경우 캐쉬되지 않는다.

  * 캐쉬된 데이터는 캐쉬의 만료일자(expired) 혹은 만료시간(max-age)이 아직 남아있는경우 의해 최신상태(fresh)인 것으로 인정된다. 최신상태일 경우 서버에 요청을 보내지않고 캐쉬를 사용한다.

  * 캐쉬가 최신 상태가 아닐(stale)경우 클라이언트는 서버에 유효성 검사(validation) 요청을 보내서 캐쉬에 존재하는 데이터를 새로 내려받지 않아도 아직 유효한지 검사를 한다. 이때 유효하지 않을경우 전체 재전송이 일어나며, 유효할경우에는 재전송이 일어나지 않으며(HTTP 304 Not modified 결과를 수신) 캐쉬의 만료일자/만료시간을 새롭게 업데이트한다.

이제 이러한 기본적인 동작들이 캐쉬 지시자들에 의해 어떤 방식으로 세부적으로 컨트롤되는지 살펴보자.

## HTTP Request Header의 캐쉬 지시자

일반적으로 캐시여부를 결정하는데 사용되는 정보인 캐쉬 지시자는 서버의 HTTP응답에 포함되어있다. 하지만 가끔씩 HTTP 요청을 보낼경우에도 캐쉬 지시자를 사용하는 경우가 있다. 프록시 서버가 중간에 있을경우, 프록시서버에 있는 캐쉬데이터가 최신(fresh)일 경우 이를 바로 클라이언트에 돌려주게된다. 하지만 캐쉬가 최신임에도 프록시가 아닌 실제서버에서 데이터를 직접 받아오고 싶은경우 `Cache control: no-cache` 필드를 지정하면 해당 요청은 프록시를 거쳐 실 서버까지 도달하게 된다.  
`Cache control: no-store` 필드가 지정될 경우에는 캐쉬가 해당 요청이나 그에대한 응답을 절대로 저장하지 않도록 한다. <sup id="fnref-405-rfc"><a href="#fn-405-rfc" rel="footnote">2</a></sup>  
이 두가지 필드외에도 다른 필드들이 있지만 요청(request) 측면에서는 의미가 없다. 하지만 응답(response)쪽에서 설정되었을때는 중요한 의미를 가지므로 더 자세한 내용은 다음 응답헤더의 캐쉬 지시자 섹션에서 더 살펴보도록 하자.

## HTTP Response Header의 캐쉬 지시자

서버에서 클라이언트에게 응답을 보낼때 캐쉬관련 정보를 알려주기위해 HTML 문서내에 캐쉬관련 정보가 포함된 meta 태그를 이용하는 방법도있지만 이방법은 지원하는 브라우저들이 한정되어있기때문에 거의 사용되지 않는다. 일반적으로는 캐쉬컨트롤을 위해 응답 헤더에 캐쉬관련 필드를 추가하는 방법많이 사용하므로, 이를 더 자세히 알아보도록 하자.

### 1. Pragma

`Pragma: cache`, 혹은 `Pragma: no-cache` 등의 Pragma를 이용할경우 특정브라우저에서만 동작하기때문에 정확하게 캐쉬컨트롤이 되는지 보장받을 수 없으므로,

HTTP/1.0에서 캐시 등을 제어하기 위해 사용되었으나 HTTP/1.1에서는 좀더 정교한 캐시컨트롤을 위해 별도의 지시자가 추가되었으므로 Pragma는 사용하지 않는것이 좋다(1.0과의 하위 호환성을 위해 남아있음).

### 2. Expires

  * 사용 예: `Expires: Fri, 30 Oct 2014 20:11:21 GMT`

HTTP date 형태로 날짜를 지정하며 로컬타임이 아닌 GMT를 사용한다.  
웹서버에서 last access time 혹은 last modified time 기준으로  absolute 시간값을 셋팅하여 response  
규칙적인 시간간격으로 컨텐츠가 바뀔때 유용하다.

  * 단점:  
      * 웹서버와 클라이언트간의 time sync가 맞지않으면 무의미.(클라이언트의 경우 임의로 시간 변경이 가능한 경우가 많기때문에 문제의 소지가 크다)
      * 한번 expires 를 설정해두고 업데이트하지 않으면, 컨텐츠가 바뀌지 않았음에도 새롭게 요청이 들어와서 로드가 커질 수 있다.

### 3. Cache-Control

  * 사용 예: `Cache-Control: max-age=3600, must-revalidate` 
      * 해석: 3600초까지는 서버에 요청없이 캐쉬데이터를 사용하며, 3600초 경과 후에는 서버에 꼭 유효성검사를 해야하며 네트웍 상황등의 이유로 유효성검사가 불가능할경우에도 절대로 캐쉬데이터를 사용하면 안된다는것을 명시.
  * 옵션: 
      * `max-age=[seconds]` — Expires와 동일한 의미지만 고정된 절대시간 값이아닌 요청 시간으로부터의 상대적 시간을 표시한다. 명시된경우 Expires보다 우선시된다.</p> 
      * `s-maxage=[seconds]` — max-age와 동일한 의미지만  shared caches (예: proxy)에만 적용됨. 명시된경우 max-age나 Expires보다 우선시된다.<sup id="fnref-405-mobi"><a href="#fn-405-mobi" rel="footnote">3</a></sup>
    
      * `public` — 일반적으로 HTTP 인증이 된 상태에서 일어나는 응답은 자동으로 private 이 되지만 public을 명시적으로 설정하면 인증이된 상태더라도 캐시하도록 한다.
    
      * `private` — 특정 유저(사용자의 브라우저)만 캐쉬하도록 설정. 여러사람이 사용하는 네트워크상의 중간자(intermediaries)역할을 하는 shared caches (예: proxy) 에는 경우 캐쉬되지 않는다.
    
      * `no-cache` — 응답 데이터를 캐쉬하고는 있지만, 일단 먼저 서버에 요청해서 유효성 검사(validation)을 하도록 강제한다. 어느정도 캐쉬의 효용을 누리면서도 컨텐츠의 freshness를 강제로 유지하는데 좋다.
    
      * `no-store` — 어떤 상황에서도 해당 response 데이터를 저장하지 않음.
    
      * `no-transform` — 어떤 프록시들은 어떤 이미지나 문서들을 성능향상을위해 최적화된 포멧으로 변환하는 등의 자동화된 동작을 하는데 이러한 것을 원치 않는다면 이 옵션을 명시해주는 것이 좋다.
    
      * `must-revalidate` —  HTTP는 특정 상황(네트워크 연결이 끊어졌을때 등)에서는 fresh하지 않은 캐쉬 데이터임에도 불구하고 사용하는 경우가 있는데, 금융거래 등의 상황에서는 이러한 동작이 잘못된 결과로 이어질 가능성이 있기 때문에 이 지시자를 통해서 그러한 사용을 방지한다.
    
      * `proxy-revalidate` — `must-revalidate`와 비슷하지만 proxy caches 에만 적용된다.

## 캐쉬 만료(cache expiration) 체크하기

Expired나 Cache-Control의 max-age 필드를 이용하여 캐쉬에 저장된 데이터가 오래된(stale)경우 서버로 재요청을 한다. 캐쉬 만료전까지는 서버에 요청할 필요가 없으므로 **네트워크 요청에 대한 왕복(roundtrip)시간을 절약**할 수 있다.

## 캐쉬 유효성 검사(cache validation) 방법

현재 캐쉬하여 갖고있는 데이터가 최신(freshness)인지 확인하여 새로 전송이 필요한지 확인하는 것을 캐쉬 유효성 검사라고 한다. 캐쉬데이터가 유효한 경우에는 서버에서 full response를 할 필요가 없어지므로 **네트워크 대역폭(bandwidth)을 절약**하게 해준다. 유효성 검사를 수행하기 위해 앞서 언급한 캐쉬 지시자나 서버에서 생성/관리하는 ETag를 사용하게 된다.

### ETag란?

컨텐트 기반의 캐쉬를 위해 특정 컨텐트에 대해 MD5 해쉬 등의 방법으로 다이제스트를 생성하면 해당 컨텐츠가 변경되었는지를 판별할수있는데, 이러한 다이제스트 값을 ETag(entity tag)로 사용한다.<sup id="fnref-405-heroku"><a href="#fn-405-heroku" rel="footnote">4</a></sup>  
대부분의 HTTP서버들은 정적인 컨텐츠(파일이나, 내용이 변하지 않는 웹페이지 등)에 대해 ETag와 Last-Modified를 생성하여 헤더에 추가하도록 설정되어있으며, 이를 HTTP서버 관리자가 원하는대로 수정하는 것이 가능하다.

클라이언트가 특정 URL을 서버에 요청을 하면  웹서버는 해당 요청에 대한 응답을 하게되는데 해당 응답 헤더에 ETag, Last-Modified 항목이 포함되어있다. 클라이언트가 동일한 URL로 다시 요청을 할 경우 클라이언트는 요청 헤더의 `If-None-Match`필드에 ETag값을 포함시켜서 보내게 되고, 서버는 클라이언트에서 보내온 ETag와 현재 컨텐츠의 ETag를 비교하여 유효성을 검사한다.  
ETag가 동일한 경우 응답 바디없이 헤더만 HTTP 304 Not modified를 리턴하고, ETag가 다른것이 발견되면 전체응답을 완전히 재전송하게된다.

### 동적인 컨텐츠는 어떻게 캐싱하나?

PHP, ASP, JSP등의 서버사이드 프로그램을 통해서 동적으로 생성된 컨텐츠는 Last-Modified, ETag, Expires, Cache-Control 등의 정보가 없어서 손쉽게 캐쉬할 수 없다.

이러한 문제점을 해결하기 위해 다음과 같은 방법들이 사용가능하다.

  1. 추천: 동적으로 생성된 컨텐츠를 파일로 저장해두고 정적인 컨텐츠처럼 취급한다. Last-Modified를 보존할 수 있도록 내용이 바뀔때만 새롭게 파일로 저장한다.</p> 
  2. 응답 헤더에 freshness를 판별할 수 있도록 캐쉬 지시자를 추가

  3. 2번으로 부족할경우 서버사이드 프로그램 자체에서 ETag 같은 validator를 생성하여 응답헤더에 추가하고, 클라이언트에서 http 요청이 올경우 이를 파싱하여 직접 validation을 한다.

<li id="fn-405-cache">
  <p>
    <a href="http://www.mnot.net/cache_docs/">http://www.mnot.net/cache_docs/</a>&#160;<a href="#fnref-405-cache" rev="footnote">&#8617;</a> </li> 
    
    <li id="fn-405-rfc">
      <a href="http://tools.ietf.org/html/rfc2616#section-14.9.3">RFC 2616</a>&#160;<a href="#fnref-405-rfc" rev="footnote">&#8617;</a>
    </li>
    <li id="fn-405-mobi">
      <a href="http://www.mobify.com/blog/beginners-guide-to-http-cache-headers/">http://www.mobify.com/blog/beginners-guide-to-http-cache-headers/</a>&#160;<a href="#fnref-405-mobi" rev="footnote">&#8617;</a>
    </li>
    <li id="fn-405-heroku">
      <a href="https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers">https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers</a>&#160;<a href="#fnref-405-heroku" rev="footnote">&#8617;</a> </fn></footnotes>