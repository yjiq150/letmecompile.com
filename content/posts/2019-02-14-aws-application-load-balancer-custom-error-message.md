---
title: AWS Application Load Balancer Custom Error Message
date: 2019-02-13T16:01:05+00:00
url: /aws-application-load-balancer-custom-error-message/
dsq_thread_id:
  - 7233203997
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps
  - AWS

---
AWS의 Application Load Balancer(LB)를 통해 실제 서버가 연결되어있을 때 서버가 죽어서 응답을 해주지 않으면 LB에서 대신 502 Gateway Error로 응답을 해준다. 이때 502 HTTP status code와 함께 LB의 디폴트 에러페이지가 응답으로 나가게된다.

개발을 하다보니 이 디폴트 에러를 보여주는 대신 customize 한 응답을 내려줘야 할 필요가 생겨서 리서치를 해보았으나, 안타깝게도 설정이 불가능 하다는 것을 확인했다. LB에서 request URL path에 따라서 리디렉션이나 포워딩 또는 정해진 스태틱 응답을 내보내는 것은 설정 가능하지만, 특정 에러코드에 대해서 에러 응답 내용을 변경하는 것은 불가능하다.

아래 링크에 가면 현재 AWS Forum에서 120명 넘게 댓글이 달려있는데 이 기능을 만들어달라고 7년째 &#8220;+1&#8221; 댓글이 달리고 있는 것을 볼 수 있다.  
2011년에 올라온 글이지만 2019년 현재까지도 불가능 해서 계속 댓글이 달리고있다.

<https://forums.aws.amazon.com/thread.jspa?threadID=72363&start=100&tstart=0>

## 부분적인 해결책

AWS에서 CDN역할을 하고있는 CloudFront 서비스를 사용하면 custom error response설정이 가능하다. 하지만 CloudFront가 필요없는 상황에서 도입하는 것은 비용 측면에서도 비효율 적이고 관리 부담이 증가하기때문에 잘 고려해서 도입해야한다.

### 결론

AWS에서 Application LB에 error response customize 기능을 어서 추가해주기를 기원합니다 (__).