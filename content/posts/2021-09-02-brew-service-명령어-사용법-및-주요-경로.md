---
title: brew service 명령어 사용법 및 주요 경로
date: 2021-09-02T14:49:54+00:00
url: /brew-service-명령어-사용법-및-주요-경로/
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/aws-application-load-balancer-custom-error-message/"     class="post-832"><span class="crp_title">AWS Application Load Balancer Custom Error Message</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/aws-application-load-balancer-custom-error-message/"     class="post-832"><span class="crp_title">AWS Application Load Balancer Custom Error Message</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 개발 환경 설정

---
## 명령어

  * brew services
      * 그냥 실행하면 현재 유저 권한으로 등록/실행된 service 목록 조회
      * sudo를 붙여서 실행하면 root로 등록/실행된 service목록 조회
      * 실수로 동일한 서비스가 root와 현재 유저에 둘다 실행되서 겹치는 경우가 있을 수 있으니 꼭 root권한이 필요한 서비스가 아니라면 현재 유저 권한 기준으로 서비스를 실행할 것
  * brew services **[restart/stop/start]** **[서비스명]**
      * 재시작/종료/시작

## 주요 경로

  * 로그 /usr/local/var/log
  * 설정 /usr/local/etc
  * httpd
      * 설정 /usr/local/etc/httpd
      * 데이터 및 로그 /usr/local/var/log/httpd
  * mysql
      * 설정 /usr/local/etc/my.cnf
      * 데이터 및 로그 /usr/local/var/mysql