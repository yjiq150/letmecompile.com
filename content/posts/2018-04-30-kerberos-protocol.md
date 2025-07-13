---
title: 커버로스 프로토콜(Kerberos Protocol) – 서버 접근 권한 관리
date: 2018-04-30T04:37:30+00:00
url: /kerberos-protocol/
dsq_thread_id:
  - 6642767206
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps

---
서버가 몇대 없고, 사용자의 숫자도 얼마 안되는 경우 각 서버별로 수동으로 유저를 추가하거나 권한을 부여해서 수동으로 관리하는것이 가능하다. 하지만 아래와 같이 점점 서버의 숫자가 많아지고, 유저들도 늘어나는 경우 관리에 드는 비용이 점점 커진다.

  * 각 서버별로 접근 가능한 사용자(=클라이언트)들의 권한을 관리해야하는 경우
  * 서버가 추가될때마다 혹은 유저가 추가/삭제 될때마다 매번 접근가능 유저를 등록하고 관리해야 하는 경우

이런 경우에는 클라이언트/서버 외에 제3의 인증서버(Authentication Server, AS)를 도입 하고, 이와 연동된 티켓 부여 서비스(Ticket Granting Service, TGS)를 통해 티켓을 발급하여 유효한 티켓이 있는 유저만 서비스 서버(Service Server, SS)에 접속을 할 수 있도록 제어하는 커버로스(Kerberos) 프로토콜 을 도입하여 편리하게 관리하는 것이 가능하다.

## Kerberos Protocol 동작 순서

![kerberos.png][1] 

참고: 아래 설명에서 encrypt(키, 데이터)는 키값으로 데이터를 암호화 한다는 뜻이다.

  1. 클라이언트가 ID/PWD 또는 공개키 등으로 클라이언트의 머신에 먼저 로그인</p> 
  2. 클라이언트가 아래 메시지를 AS로 전송
    
      * User ID (암호화 되지않은 일반 텍스트)
  3. AS는 User ID가 DB에 존재하는지 찾아본 후 존재하는 경우 아래 두 메시지를 리턴 
      * encrypt(key: 클라이언트 PWD기반 비밀키, data: &#8220;TGS 세션키&#8221;)
      * 티켓을 발급받을 수 있는 티켓(Ticket-Granting-Ticket, TGT) = encrypt(TGS 비밀키, &#8220;Client ID, 주소, 유효기간, TGS 세션 키&#8221;)
  4. 클라이언트는 아래 두 메시지를 TGS로 전송 (TGS 세션키는 수신한 메시지를 복호화해서 획득) 
      * Authenticator = encrypt(key: TGS 세션키, data: &#8220;Client ID, timestamp&#8221;)
      * TGT (클라이언트는 TGS 비밀키를 모르기때문에 복호화 또는 데이터 조작 불가)
  5. TGS는 전달받은 TGT(TGS 비밀키로 복호화 가능), 암호화된 Authenticator (TGS 세션키로 복호화 가능)를 모두 복호화 해서 안에 담긴 Client ID가 일치하는지 확인하여 일치할 경우 아래 두 메시지를 리턴 
      * encrypt(key: TGS 세션키, data: SS 세션키)
      * Ticket = encrypt(key: SS 비밀키, data: &#8220;Client ID, 주소, 유효기간, SS 세션키&#8221;)
  6. 클라이언트는 아래 두 메시지를 SS로 전송 
      * Authenticator = encrypt(key: SS 세션키, data: &#8220;Client ID, timestamp&#8221;) 
      * Ticket
  7. SS는 전달받은 Ticket, Authenticator를 복호화 해서 안에 담긴 Client ID일치 확인 후 일치할 경우 아래 메시지를 리턴 
      * encrypt(key: SS 세션키, data:Authenticator안에 담겨있던 timestamp)

  8. 클라이언트는 전달받은 timestamp와 자신이 Authenticator에 담아보냈던 timestamp의 값이 일치하는 확인 후, 일치할경우 실제 작업을 시작한다.

## Kerberos Protocol 단점

  * 커버로스 서버가 SPOF(Single Point Of Failure) 이기 때문에 이 서버가 다운되면 기존에 이미 로그인된 유저를 제외한 새롭게 로그인을 시도하는 유저가 서버에 접속하는것이 불가능해진다.
  * AS, TGS, SS간에 서로 비밀키를 미리 알고있어야 하기때문에 동기화 이슈가 존재한다.
  * 인증이 한번 완료되면 유효기간이 존재하긴 하지만 티켓이 클라이언트에 보관되므로 티켓이 탈취될 가능성이 있다.

 [1]: /uploads/2018/04/kerberos.png