---
title: 도메인 설정 변경 확인 명령어
author: yjiq150
type: post
date: 2018-01-28T14:58:48+00:00
url: /verify-domain-setting-changes/
dsq_thread_id:
  - 6443295187
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 프로그래밍 기타
  - 웹 개발
  - DevOps

---
도메인 레코드를 추가한다거나, 네임서버 주소를 바꾼다거나 하는 등 도메인 관련 설정을 변경하고나면 제대로 설정이 전파되었는지 확인하기 위해 지루하게 기다리는 시간이 이어진다.  
도메인 설정이 모든 DNS들에게 전파되는데는 최대 1~2일까지 걸린다고 하지만 요즘은 refresh time이 빨라서 그런지 느낌상 대다수가 거의 1시간이내로 변경되긴 하는 듯 하고, 특히나 국내에서만 보면 10분내로도 변경된 설정이 바로바로 반영이되는듯 하다.

Tip: 해외에서 웹사이트가 잘 접속되는지 확인하려면 [https://pulse.turbobytes.com][1] 사이트를 추천한다.

## DNS 변경내역 확인을 위한 더 정확한 방법

A레코드 같은것들이야 변경되면 웹브라우저를 열어 해당 도메인에 요청을 계속 보내거나 ping을 해서 IP주소가 잘 변경되었는지 확인하면 되지만 그외 레코드들은 확인할 방법이 마땅치 않아서 이것저것 찾아보던 중 다음 명령어를 발견했다.

    dig +trace letmecompile.com ANY
    

  * dig 명령어는 Linux나 Mac OS X에 기본적으로 설치되어있다. 
  * `+trace`옵션은 DNS query가 어떤 순서로 일어나는지 중간과정까지 보여준다
  * NS, A, TXT, MX, CNAME 등의 레코드 정보를 조회 가능하고 `ANY` 라고 입력하면 모든 레코드 정보를 다 보여준다
  * example.com에 대해서만 조사할경우 subdomain정보들은 나타나지 않는다. 따라서 admin.example.com 같이 sub domain을 사용하는 경우 `dig admin.example.com ANY` 처럼 같이 서브도메인에 대해 따로 query를 해야한다.

## 도메인 쿼리 순서

아래 결과를 보면 `letmecompile.com` 도메인에 해당하는 IP주소를 찾기위해 어떤 순서로 query가 일어나는지까지 같이 확인 할 수 있어서 DNS resolve 과정에서 문제가 발생하고있는 경우 해당 문제가 되는 계층을 찾는데도 도움이 될 수 있다.

  1. 루트 NS (Name Server, 네임서버) 에 조회
  2. .com 주소이므로 루트 NS가 TLD(Top Level Domain) NS 주소를 알려준다
  3. TLD NS는 다시 letmecompile.com에 연결된 ns.hosting.co.kr NS 주소를알려준다
  4. ns.hosting.co.kr NS는 letmecompile.com에 관련된 상세한 도메인 레코드 정보들을 모두 알려준다

    ; <<>> DiG 9.9.7-P3 <<>> +trace letmecompile.com any
    ;; global options: +cmd
    .           210183  IN  NS  f.root-servers.net.
    .           210183  IN  NS  j.root-servers.net.
    .           210183  IN  NS  d.root-servers.net.
    .           210183  IN  NS  c.root-servers.net.
    .           210183  IN  NS  g.root-servers.net.
    .           210183  IN  NS  e.root-servers.net.
    .           210183  IN  NS  i.root-servers.net.
    .           210183  IN  NS  l.root-servers.net.
    .           210183  IN  NS  a.root-servers.net.
    .           210183  IN  NS  m.root-servers.net.
    .           210183  IN  NS  k.root-servers.net.
    .           210183  IN  NS  h.root-servers.net.
    .           210183  IN  NS  b.root-servers.net.
    ;; Received 447 bytes from 168.126.63.1#53(168.126.63.1) in 2 ms
    
    com.            172800  IN  NS  k.gtld-servers.net.
    com.            172800  IN  NS  e.gtld-servers.net.
    com.            172800  IN  NS  d.gtld-servers.net.
    com.            172800  IN  NS  j.gtld-servers.net.
    com.            172800  IN  NS  l.gtld-servers.net.
    com.            172800  IN  NS  g.gtld-servers.net.
    com.            172800  IN  NS  i.gtld-servers.net.
    com.            172800  IN  NS  a.gtld-servers.net.
    com.            172800  IN  NS  b.gtld-servers.net.
    com.            172800  IN  NS  f.gtld-servers.net.
    com.            172800  IN  NS  c.gtld-servers.net.
    com.            172800  IN  NS  h.gtld-servers.net.
    com.            172800  IN  NS  m.gtld-servers.net.
    com.            86400   IN  DS  30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
    com.            86400   IN  RRSIG   DS 8 1 86400 20180210050000 20180128040000 41824 . TZQQWbFGU96SorpfPjl9GLxI2XIyODoYRMMUOVMIJIeljgajAg6kiF5k QIhjzwwgLLMsBtIi23muyD7l7wyuwpfcR4+XBAWEUPCMaV5qXJmubxxX fr0Iwiw0r7ZrzzPkR/lWgdrDTZzZly/rESPTQBoiPP+JQo8zGZZ/cuv0 xvJc+WhBSaaFZbDCp3VGlT3BDkXbptzlQs67TXjnfVMYdqhmx1PtuDUT loDCmySqV0Lq2fYkcp8qDtHG+EVLTlK7x2843QuTXYZDcLiGtn9H7otw k+/xuigodi6St3fljR22JCJdpNdLGxPlWrc7+XLmTuWlJ67h4ecy4azG LxNpaA==
    ;; Received 1173 bytes from 192.33.4.12#53(c.root-servers.net) in 180 ms
    
    letmecompile.com.       172800  IN  NS  ns2.hosting.co.kr.
    letmecompile.com.       172800  IN  NS  ns1.hosting.co.kr.
    CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q1GIN43N1ARRC9OSM6QPQR81H5M9A NS SOA RRSIG DNSKEY NSEC3PARAM
    CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 8 2 86400 20180201054813 20180125043813 46967 com. po7AuPhC1O3zHl6hE50ynphnyjk+eJWo+Ovi+MQ5WurNfbz70uTlich+ gKO4lAs6tWGp+1t9eNkmdlf5gy2Brnsuytoh6Jao0m7ts59dIaYeNdiN xzAqOQO9QoBmkxazrTJvxF+o/RZwQSo3SK9yLHy3DqVfayqrCMQfhqcy 5js=
    GBPUGJD7VK1OTQ5CGULTKGOPKJS34E6A.com. 86400 IN NSEC3 1 1 0 - GBQ0DEQ68SC0E4OEIOJ5EATRSIK8JSKM NS DS RRSIG
    GBPUGJD7VK1OTQ5CGULTKGOPKJS34E6A.com. 86400 IN RRSIG NSEC3 8 2 86400 20180202053407 20180126042407 46967 com. apy6ZhvAEDUDiDigYASbaAN/TSUYc7uUE7ycYv8wKa6CFOntveE6B28W Ei7x7z7B054keMGMAC64vcZTYpqICdpQ0eBUTImfD8WNqAlwGsEx/rV+ YKtjaBqoSTSDwej/5Jdc++WZEbh20XAXuefKzs4K1rxYlftnGLvMUiTh 45U=
    ;; Received 576 bytes from 192.54.112.30#53(h.gtld-servers.net) in 142 ms
    
    letmecompile.com.       180 IN  SOA ns1.hosting.co.kr. admin.letmecompile.com. 11 21600 1800 1209600 180
    letmecompile.com.       180 IN  A   13.124.168.202
    letmecompile.com.       180 IN  MX  10 ASPMX.daum.net.
    letmecompile.com.       180 IN  MX  20 ALT.ASPMX.daum.net.
    letmecompile.com.       180 IN  NS  ns1.hosting.co.kr.
    letmecompile.com.       180 IN  NS  ns2.hosting.co.kr.
    letmecompile.com.       180 IN  TXT "v=spf1 include:_spf.daum.net ~all"
    ;; Received 277 bytes from 110.45.158.3#53(ns2.hosting.co.kr) in 6 ms

 [1]: https://pulse.turbobytes.com/