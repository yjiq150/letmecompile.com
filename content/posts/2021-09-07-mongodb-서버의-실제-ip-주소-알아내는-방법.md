---
title: mongodb 서버의 실제 IP 주소 알아내는 방법
date: 2021-09-07T09:05:55+00:00
url: /mongodb-서버의-실제-ip-주소-알아내는-방법/
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Database
  - Server

---
mongodb+srv://mongodb-atlas-serverles.asdf.mongodb.net

위와같은 mongodb connection string이 있을때 해당 mongodb에 접속할 수 있는 실제 서버의 IP주소를 알고싶은 경우 다음과 같이하면 된다.

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># -type=SRV 옵션을 사용해서 DNS 레코드 중 SRV 타입을 검색한다
# connection string에 명시된 주소 앞쪽에 _mongodb._tcp 를 붙여준다.
$ nslookup -type=SRV _mongodb._tcp.mongodb-atlas-serverles.asdf.mongodb.net

Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
_mongodb._tcp.mongodb-atlas-serverles.asdf.mongodb.net	service = 0 0 27017 mongodb-atlas-serverless-example-dev-lb.asdf.mongodb.net.</pre>

출력된 결과를 보면 mongodb-atlas-serverless-example-dev-lb.asdf.mongodb.net 부분이있는데 이것이 실제 주소이다.

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># 위에서 얻은 주소로 다시한번 쿼리
$ nslookup mongodb-atlas-serverless-example-dev-lb.asdf.mongodb.net

Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
mongodb-atlas-serverless-example-dev-lb.dvfze.mongodb.net	canonical name = 123456-nlb-12345.elb.ap-southeast-1.amazonaws.com.
Name:	123456-nlb-12345.elb.ap-southeast-1.amazonaws.com
Address: 13.213.131.24
Name:	123456-nlb-12345.elb.ap-southeast-1.amazonaws.com
Address: 13.213.229.91</pre>

여기서 나온 IP 주소들이 실제 mongodb 서버의 주소이다.