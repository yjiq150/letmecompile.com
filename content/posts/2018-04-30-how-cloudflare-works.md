---
title: 클라우드플레어(Cloudflare) 동작 원리
date: 2018-04-30T04:40:15+00:00
url: /how-cloudflare-works/
dsq_thread_id:
  - 6642769183
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps

---
**&#8220;클라우드플레어(Cloudflare)를 적용만 하면 당신의 사이트의 속도를 빠르게 할 수 있다&#8221;** 라는 클라우드플레어 광고 문구를 보고 도대체 클라우드플레어가 중간에서 어떤일을 하는지 궁금해서 조금 자세히 찾아보았다.

## 사이트 접속시에 어떤 일이 일어날까?

사용자가 http://example.com 주소를 입력했을때 일반적으로는 DNS에 example.com을 조회하면 그에 해당하는 서버의 IP주소를 알려주고, 해당 IP주소로 실제 HTTP요청이 일어난다. 하지만 클라우드플레어(Cloudflare)를 사용하게되면 모든 요청이 클라우드플레어 서버를 먼저 한번 거친 후, 필요한 경우에만 실제 서버의 IP주소까지 요청이 도달한다.

  * 일반적인 접근: DNS -> Server IP
  * 클라우드플레어 설정 후: DNS -> Cloudflare Server(CDN) -> Your IP (Origin)

![][1] 

일반적인 경우 자주 바뀌거나, 유저 별로 다르게 보여줘야하는 동적인 컨텐츠(dynamic content)들에 대한 요청은 CDN(Content Delivery Network)을 거치지 않고 직접 오리진으로 향하게 한다. 대신 해당 컨텐츠안에 링크로 포함된 JS, CSS, 이미지 파일 요청 등은 정적 컨텐츠(static content)이기 때문에 잘 변하지 않으므로 별도의 CDN을 설정해서 제공한다.

하지만 특이하게도 클라우드플레어가 설정된 경우 사이트로 향하는 모든 traffic이 정적/동적 컨텐츠와 무관하게 클라우드플레어 서버를 일단 한번 거치도록 되어있다. 이로 인해서 다음과같은 다양한 장/단점들이 생겨난다.

## 클라우드플레어의 장점

  * CDN을 별도로 설정해줄 필요 없이 네임서버 설정 변경만으로 모든 컨텐츠를 CDN을 통해 전달이 가능하다. 
      * 파일 확장자들을 기준으로 정적인 컨텐츠들은 origin에서 한번만 로드한 후 캐시해서 자동으로 CDN처럼 동작한다 (커스텀하게 규칙을 설정이 가능하다.) HTML은 캐싱하지 않는다.
      * 안타깝게도 동적인 컨텐츠의 주소들만 따로 분류해서 클라우드 플레어를 거치지 않도록 따로 설정하는것은 불가능하다.
  * 클라우드플레어 서버를 거칠때 악성 트래픽 필터링, DDoS 완화 등의 작업을 할 수 있게되어 실제 서버로 들어오는 부하를 많이 줄일 수 있다. 예를들어 의심되는 트래픽에 대해서 자동입력 방지 문자 등을 보여주고 이 부분을 통과할 경우에만 정상적으로 요청을 처리해 준다던가 하는 식으로 중간 차단을 해준다.
  * 실제 서버는 클라우드플레어 서버의 뒤쪽으로 숨겨져있기 때문에 IP를 완전히 숨길 수 있어서 추가적인 보안상의 이점이 있다.
  * 일반적으로 생각했을때 동적인 컨텐츠의 경우 캐시가 불가능하기 때문에 직접 서버로 연결하지 않고 클라우드플레어 서버를 한번 거쳐서 오게되면 속도가 더 느려질 것으로 보인다. 하지만 클라우드플레어가 전세계에 운영중인 Edge들과 해당 Edge들을 이어둔 전용 네트워크 회선을 사용하면, 실제 인터넷 상에서 엔드투엔드(End-to-End)로 가기위해 거쳐야 할 &#8220;hops&#8221; 수가 훨씬 줄어든다고 클라우드플레어 측에서 설명 하고 있다. 이 부분은 네트웍 상황에따라 장점이 될 수도있고 단점이 될 수도 있는 부분이라 자신의 상황에 맞게 사용해야 할 것 같다. (특히 한국의 경우 엔터프라이즈 유저가 아니면 [한국 Edge 사용 불가][2]하기 때문에 국내 사용자들이 접속시에 오히려 속도가 더 느려질 수도 있으니 주의해야 한다.)

## 클라우드플레어의 단점

  * 정적인 컨텐츠 경우에는 일반적인 CDN과 동일하게 속도가 매우 빠르지만, 동적인 컨텐츠의 경우 클라우드플레어서버를 통해 실제 서버(origin) 에서 컨텐츠를 가져와야 한다. 앞서 장점쪽에서 언급했듯이 네트워크 환경에따라 이 부분이 더 빠를때도 있지만 항상 그렇다는것을 보장 할수 없다. 때문에 페이지가 열리는 속도와 직결되는 TTFB(Time To First Byte)가 느려질 가능성이 충분히 존재하고, [TTFB가 느려질수록 구글 등의 검색엔진의 SEO에서 페널티][3]를 받을 가능성도 있기때문에 주의해서 사용해야 한다.
  * 서버로 들어오는 모든 요청이 클라우드플레어를 통해서 전달되기때문에 정확한 액세스 로그기록이 힘들어진다. 때문에 특정 사용자의 기록을 정확히 찾아내서 문제점을 파악한다거나 할 때 좀더 어려움을 겪을 수 있다.

## Tip: 클라우드플레어의 CDN기능을 끄고 네임서버로만 활용하기

클라우드플레어의 네임서버를 사용해보니 클라우드플레어의 네임서버(RTT 10ms 내외)는 국내 호스팅 또는 도메인 등록 대행 업체들이 제공하는 네임서버 (RTT 200ms 내외) 에 비해 훨씬 안정적이고 속도가 빠른편이다. AWS의 Route53을 이용해도 안정적이고 빠르지만 도메인 하나당 월 0.5달러를 지불해야해서 은근한 부담이 되는데, 클라우드플레어의 모든 CDN 기능을 비활성화 하고 네임서버로만 사용하면 이부분을 무료로 해결할 수 있어서 매우 좋은 것 같다. (클라우드플레어 네임서버로 설정을 한 후, 클라우드 플레어 설정페이지로 이동하여 연결된 도메인들의 구름모양 아이콘을 모두 비활성화하면 된다.

### 참고 자료

  * [클라우드플레어 Network DataCenter 위치][4]
  * [클라우드플레어 동작 방식][5] 
  * [How Cloudflare divert malicious traffic][6]
  * [클라우드플레어에서 캐시하는 파일 확장자][7]

 [1]: https://support.cloudflare.com/hc/en-us/article_attachments/201742508/overview.png
 [2]: http://ryush00.tistory.com/448
 [3]: https://moz.com/blog/improving-search-rank-by-optimizing-your-time-to-first-byte
 [4]: https://www.cloudflare.com/network/
 [5]: https://support.cloudflare.com/hc/en-us/articles/205177068-Step-1-How-does-Cloudflare-work-
 [6]: https://www.quora.com/How-does-Cloudflare-work-Does-CloudFlare-just-divert-malicious-traffic
 [7]: https://support.cloudflare.com/hc/en-us/articles/200172516-Which-file-extensions-does-Cloudflare-cache-for-static-content-