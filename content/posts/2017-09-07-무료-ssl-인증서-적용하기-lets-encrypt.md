---
title: 무료 SSL 인증서 적용하기 (Let’s Encrypt)
author: yjiq150
type: post
date: 2017-09-07T01:04:08+00:00
url: /무료-ssl-인증서-적용하기-lets-encrypt/
dsq_thread_id:
  - 6125084961
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발

---
plain http 통신을 사용하는 경우 로그인 등의 인증 요청을 할때 plain text가 바로 노출이 되기때문에 항상 보안위험이 존재한다. 이를 막기 위해서 SSL이 적용된 https 통신을 하게되는데, 이를 위해서는 SSL 인증서가 필요하다. 일반적으로 여러 인증기관에서 웹서버용 SSL 인증서를 판매하고있는데 비용이 꽤 들다보니 기업들은 많이들 구입하여 사용했지만, 소규모 웹사이트나 개인 웹사이트들의 경우 비용 문제로 인증서를 구입하지 않고 대다수가 http를 사용하고 있는 상황이다.  
하지만 Let&#8217;s Encrypt 서비스를 이용하면 비싼 돈을주고 SSL용 인증서를 구입하지 않고도 무료로 SSL인증서를 발급 받을 수 있으며, 심지어 자동화된 툴을 이용하여 더욱 편리하게 인증서 발급 및 서버 설정까지 가능하다.

(Let&#8217;s Encrypt의 경우 IdentTrust와 교차서명인된 인증서를 발급받을 수 있는 CA(Certificate Authority)이고, 이 인증서는 모든 주요 브라우저에서 문제없이 사용될 수 있다.)

그러면 실제 Ubuntu + Apache 환경에서 어떤식으로 설치하고 사용할 수 있는지 알아보도록 하자.

## Let&#8217;s Encrypt 자동화 툴 설치

더 자세한 내용은 https://certbot.eff.org/#ubuntutyakkety-apache 에 나와있다.

    sudo apt-get update
    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install python-certbot-apache 
    

## 실행 전에 확인할 사항

하나의 서버에서 apache의 vhost(virtual host, 가상호스트) 기능을 이용하여 여러개의 웹사이트를 운영하고 있다면 각각의 도메인별로 인증서를 따로 발행하는 것이 좋다. 이러한 상황에서 letsencrypt 자동화 스크립트가 잘 작동하려면 vhost 설정이 다음 조건을 만족해야 한다.

> `/etc/apache2/sites-available` 폴더에 각 사이트당 1개의 .conf 파일이 작성되어있어야 한다. 

  * 표준화된 설정방식은 `/etc/apache2/sites-available`에 example.conf 파일을 추가 후에 `/etc/apache2/sites-enabled`에 해당 파일을 심볼릭 링크를 걸어주는 것이다. 하지만 귀찮아서 `/etc/apache2/sites-available`에는 파일을 생성하지 않고 `/etc/apache2/sites-enabled`에 직접 파일을 생성하는 경우가 있는데, 이 경우 certbot 스크립트가 제대로 인식하지 못하게 되니 주의하자.
  * example.conf 하나의 파일에 사실 여러개의 vhost를 한번에 지정하는 방식도 많이들 사용하는데 이렇게 된 경우에도 certbot에서 제대로 인식을 하지 못한다. 따라서 하나의 파일에 여러개의 vhost가 지정되어 있다면, 귀찮더라도 여러개의 파일로 쪼개도록 하자.

## Let&#8217;s Encrypt 인증서 발급 및 자동 서버 환경 설정

위의 &#8220;실행 전 확인 사항&#8221; 단계를 무사히 끝마쳤으면 다음 명령어만 실행하면 된다.

    sudo certbot --apache
    

위 명령어를 넣으면 certbot 설정 UI가 나타난다.

  1. ServerName 선택 
      * Virtual host에 정의된 ServerNames 항목들이 보여지고 이 중 인증서를 발급하고 적용할 것을 고를 수 있다. 
      * example.com 이라는 사이트가 example.conf 에 정의되어있는 경우, 차후 인증서 발행이 완료될때 example-le-ssl.conf 라는 SSL을 위한 conf 파일이 자동으로 생성된다.
      * 주의: 여러 사이트를 운영중이라면 한번에 하나만 선택하도록 한다. 여러사이트를 한번에 선택해 버리면 여러 사이트들이 동일한 SSL 인증서가 적용되어버리는 문제점이 존재하기 때문이다. (이경우 인증서에 포함된 도메인과 실제 사이트의 도메인이 달라져버리게 된다.)
  2. Easy / Secure 모드 선택 
      * Easy를 선택하는 경우 필요에 따라 http, https 각각 개별적으로 사용가능하다.
      * Secure를 선택하면 경우 도메인에 해당하는 모든 http로 들어온 요청을 https로 redirect 시켜주도록 아파치가 설정된다. (추천)

두가지 선택이 끝난 후에는 알아서 인증서를 발급하고, 각 vhost별로 SSL 설정을 완료한 후 apache까지 재시작 해준다.  
(DNS record에 등록된 서버 IP를 이용해서 자동으로 해당 도메인의 소유주 유효성 검사가 완료된다.)

이제 바로 웹브라우져로 사이트를 열고 https가 잘 적용되었는지만 확인하면 된다. 혹시 custom URL rewrite 기능

## Let&#8217;s Envcypt 인증서 유효기간과 자동 리뉴얼

일반적인 SSL인증서의 경우 기업 자체를 확인하여 인증서를 발행하지만, letsencrypt 인증서의 경우 단순히 도메인 소유권을 기반으로 해당 인증서를 발행하게 되므로 기간이 유효기간이 발급일로부터 3개월 밖에 되지 않는다. 따라서 이를 자동으로 리뉴얼 하는 프로세스를 만들어 둬야 한다.

아래 명령어를 통해서 실제 리뉴얼을 하기전에 문제가없는지 확인할 수 있다.

    sudo certbot renew --dry-run --agree-tos
    

dry run을 통해서 별 문제 없음이 확인되면, 바로 아래 커맨드를 날려서 실제 리뉴얼을 진행한다 (certbot이 자동으로 apache를 재시작해줌)

    sudo certbot renew
    

실제 리뉴얼은 인증서가 expire 될 날이 얼마 남지 않았을때만 발생하기 때문에 기간이 많이 남은 인증서의 경우 아래와같은 메시지를 출력하면서 실제 리뉴얼 진행은 되지 않는다.

    Processing /etc/letsencrypt/renewal/example.conf
    The following certs are not due for renewal yet:
      /etc/letsencrypt/live/example/fullchain.pem (skipped)
    

자동화된 리뉴얼을 하려면 위의 리뉴얼 명령어를 스크립트로 만든 후 `cron`을 이용해서 매일 한번씩 확인하도록 추가하면된다.

    sudo crontab -e 
    

아래 내용을 넣고 저장하면 매일 0시 0분에 리뉴얼 스크립트를 실행하고 그 output을 certbot_renewal.log에 기록한다.

    0 0 * * * certbot renew > /var/log/certbot_renewal.log
    

## API Rate limit

Let&#8217;s Encrypt의 경우 비영리 조직이라서 그런지 다양한 제약이 많이 있다. 발급과정에서 잘못된 요청을 몇번해서 불필요하게 재시도를 많이 하다보면 limit에 걸리는 경우가 있으니 주의하도록 하자.

  * 도메인별 주당 20회 인증서 발급
  * 동일 IP에서 3시간동안 10개의 어카운트 생성가능
  * 기타 등등 더 자세한 내용은 공식 문서를 참조<sup id="fnref-682-rate"><a href="#fn-682-rate" class="jetpack-footnote">1</a></sup>

<li id="fn-682-rate">
  Let&#8217;s Encrypt rate limits https://letsencrypt.org/docs/rate-limits/&#160;<a href="#fnref-682-rate">&#8617;</a> </fn></footnotes>