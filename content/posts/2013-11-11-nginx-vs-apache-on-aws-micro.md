---
title: NginX vs. Apache on AWS micro
date: 2013-11-10T15:51:56+00:00
url: /nginx-vs-apache-on-aws-micro/
dsq_thread_id:
  - 1953572854
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/ec2-snapshot-%ec%a3%bc%ea%b8%b0%ec%a0%81%ec%9c%bc%eb%a1%9c-%eb%b0%b1%ec%97%85%ed%95%98%ea%b8%b0/"     class="post-830"><span class="crp_title">EC2 Snapshot 주기적으로 백업하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/ec2-snapshot-%ec%a3%bc%ea%b8%b0%ec%a0%81%ec%9c%bc%eb%a1%9c-%eb%b0%b1%ec%97%85%ed%95%98%ea%b8%b0/"     class="post-830"><span class="crp_title">EC2 Snapshot 주기적으로 백업하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - 워드프레스 개발
tags:
  - 워드프레스 aws
  - aws micro
  - aws 마이크로
  - nginx apache 비교

---
아파치(Apache)와 엔진엑스(NginX)의 특성들을 비교한 글들은 많이 있으니 여기서 다루지 않고, 실제 AWS EC2 micro에 적용했을때 어떤 차이를 보였는지만 눈으로 쉽게 볼 수있도록 이 글을 작성한다. 본 테스트가 진행된 웹서버에서는 워드프레스 기반의 사이트가 운영되고 있으며 일간 페이지뷰는 800~1000뷰 정도 된다.

처음에 아파치를 설치해서 사용할 떄는 CPU점유율이 5분이상 100%를 넘을때가 너무 많았다. 마이크로 인스턴스의 특성상 CPU 사용률 100%가 일정시간 이상 지속되면 먹통이 되는데 덕분에 하루에 4~5회정도 사이트 접속이 불가능한 시간들이 있었다. 서버를 스몰 인스턴스(small instance)로 업그레이드 하긴 좀 아까워서 웹서버를 엔진엑스, php-fpm조합으로 교체하여 테스트 해 본 결과 매우 성공적인 성능 향상을 가져왔다.

아래 AWS 마이크로에서 각각 1주일씩 엔진엑스와 아파치를 실행했을때 CPU사용량을 비교한 그래프를 보면 성능 차이를 확연하게 볼 수 있다.

코어가 많지 않은 서버에서 웹서버를 돌린다면 아파치보다는 역시 엔진엑스를 사용하는 것이 훨씬 효율적이라는 것이 증명되었다.

[<img loading="lazy" width="1011" height="585" src="/uploads/2013/11/EC2_Management_Console.png" alt="엔진엑스, 아파치 비교"  class="alignnone size-full wp-image-237" />][1]

사용된 Apache의 주요설정은 다음과 같다.(설치 기본값에서 변경한 값만 명시됨)

    Timeout 30
    KeepAlive On
    MaxKeepAliveRequests 100
    KeepAliveTimeout 5
    

사용된 NginX의 주요 설정은 다음과 같다.(설치 기본값에서 변경한 값만 명시됨)

    worker_processes 1;
    events {
            worker_connections 1024;
    }
    http {
        sendfile on;
        client_max_body_size 30M;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 30;
        types_hash_max_size 2048;
    
        gzip on;
        gzip_disable "msie6";
        gzip_comp_level 5;
        gzip_min_length 500;
        gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    }

 [1]: /uploads/2013/11/EC2_Management_Console.png