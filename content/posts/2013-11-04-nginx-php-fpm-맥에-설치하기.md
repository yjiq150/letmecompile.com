---
title: NginX, PHP-FPM 맥에 설치하기
date: 2013-11-03T19:00:41+00:00
url: /nginx-php-fpm-맥에-설치하기/
dsq_thread_id:
  - 1933104373
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - 워드프레스 개발
tags:
  - nginx for mac
  - mac brew
  - nginx build

---
맥용 패키지 설치 프로그램인 홈브루(Homebrew)가 설치되어 있다는 것을 가정하에 진행한다.  
홈브루가 없을경우 <http://brew.sh/> 에서 다운로드하여 설치한다.

홈브루가 설치되어있을경우 다음 명령어를 통해 홈브루 포뮬러들을 업데이트 해두는 것이 좋다.

> brew update

## NginX 설치

> brew install nginx

NginX 설치를 위해서는 pcre 라이브러리가 필요한데, 디렉토리 권한문제로 다음과같은 에러가 날때가있다.

    Warning: Could not link pcre. Unlinking...
    Error: The `brew link` step did not complete successfully
    The formula built, but is not symlinked into /usr/local
    You can try again using `brew link pcre'
    

이경우 다음과같이 해결해주면 된다.

> sudo chmod 777 /usr/local/bin  
> brew link pcre  
> sudo chmod 755 /usr/local/bin

보안을 위해 777로 쓰기권한 줬던것을 설치 후 바로 원래대로 돌려놓아야 하는것에 주의한다.

homebrew를 통해 설치가 완료되면 기본적으로 웹서버 포트가 8080으로 설정이 되어있다. 이는 homebrew 설치가 현재 로그인된 사용자의 권한으로 이루어지기 때문에 1024번 미만의 포트를 사용하지 못하기 때문이다. (root 사용자만이 1024번보다 작은 포트를 사용 가능)

  * 주의: 때문에 80번 포트를 사용하고 싶다면 추후 서버포트를 80번으로 변경 후 NginX를 실행할때 `sudo`를 이용하여 root권한으로 실행하여야 한다.

로그인시 자동실행 되게 하려면 다음과 같이 명령어를 실행한다.

> cp /usr/local/Cellar/nginx/1.2.7/homebrew.mxcl.nginx.plist /Library/LaunchAgents/  
> launchctl load -w /Library/LaunchAgents/homebrew.mxcl.nginx.plist

## NginX 관련 디렉토리

다음 명령어를 통해서 NginX 관련파일들이 어디에 위치하고있는지 알 수 있다.

> brew list nginx

`/usr/local/Cellar/nginx`: NginX 관련 실행파일 및 로그, 도큐먼트 루트 위치  
`/usr/local/Cellar/nginx/1.2.7/html/`: 도큐먼트 루트

`/usr/local/etc/nginx`: 설정파일들이 위치

`/usr/local/sbin`: NginX 원본 실행파일에 대한 심볼릭 링크 존재. 따라서 이 경로에 PATH지정을 해주면 NginX 를 어떤 경로에서든 실행 가능하다.

## NginX 기본 명령어

위에서 말했듯이 80번 포트로 실행하려면 앞에 sudo를 붙여준다.

서버 시작

> nginx

서버 종료

> nginx -s stop

서버 재시작

> nginx -s reload

## PHP-FPM 설치하기

php모듈을 포함한 상태로 실행되는 아파치와는 달리, NginX는 php관련 처리할 일이 생기면 독립적인 FastCGI 프로세스로 전달하게된다. 따라서 PHP-FPM(FastCGI Process Manager) 옵션이 켜진상태로 컴파일 된 PHP가 필요하다. 따라서 OS X에 기본적으로 설치되어있는 php가 있지만, NginX와 연동이되도록 새롭게 PHP를 받아서 컴파일하는 과정이 필요하다.

일단 php설치를 위해 홈브루에 탭을 추가한다.

> brew tap josegonzalez/homebrew-php  
> brew tap homebrew/dupes

`php54` 에서 버전번호는 원하는대로 수정하면 된다.

> brew install &#8211;without-apache &#8211;with-fpm &#8211;with-mysql php54

로그인시 자동실행을 위해서는 다음 명령어를 실행한다.

> sudo cp `` `brew --prefix php54 ``/homebrew-php.josegonzalez.php54.plist /Library/LaunchAgents/
> 
> sudo launchctl load -w /Library/LaunchAgents/homebrew-php.josegonzalez.php54.plist

## NginX 설정 팁

웹서버 설정의 케이스바이 케이스다보니 경우 정답이라고 할만한 것이 없기때문에, 다양한 설정들을 참고하고, 운영중인 서버 환경에 맞게 테스트해가면서 조정하는것이 최선이다. 처음에는 워낙 막막하다보니 다음 글들을 참고하면 도움이 좀 된다.

[생활코딩 오픈튜토리얼 NginX설정][1]

[AWS EC2용 c1.medium ~ xlarge instances 샘플 설정][2]

[NginX setting for WordPress][3]

## .htaccess 를 NginX 용으로 변환하기

아파치에서 사용중이던 `.htaccess`파일이 있을경우 다음사이트를 이용하면 간단하게 NginX 문법으로 변경가능하다. <http://winginx.com/htaccess>

## 워드프레스용 NginX 서버 설정 예시

아파치에서는 .htacess를 이용하여 워드프레스의 고유주소(permalink)방식을 구현한다. 이와 비슷하게 NginX에서는 서버설정의 location 항목을 이용하여 url rewrite를 할 수 있고, 다음과 같이 설정해주면 워드프레스의 고유주소가 잘 동작한다.

        ### NginX setting for LetMeCompile.com
        server {
            listen       80;
            server_name  www.letmecompile.com letmecompile.com;
            root   /var/www/letmecompile.com;
    
            index  index.php;
    
            # rewrite rule for WordPress
            location / {
                try_files $uri $uri/ /index.php?$args;
                }
    
            location ~ \.php(?:/|$) {
                fastcgi_pass   unix:/var/run/php5-fpm.sock;
                #fastcgi_pass   127.0.0.1:9000; 
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                include fastcgi_params;
            }
    
            location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                expires 1y;
            }
            # deny access to .htaccess files, if Apache's document root
            # concurs with NginX's one 
            location ~ /\.ht {
               deny  all;
            }
        }
    

## 팁: 페이지 로딩속도 측정

물론 siege나 nGrinder, Bees with Machine Guns 등을 사용하여 전문적인 부하 테스트(Load test)를 해야 정확한 결과가 나오겠지만, 간편하게 구글 페이지 스피드로 웹서버 셋팅 변경 후 한번씩 페이지로딩 속도를 점검 해보는 것도 좋다.

<http://developers.google.com/speed/pagespeed/insights/>

 [1]: http://opentutorials.org/module/384/4530
 [2]: https://gist.github.com/nateware/3988974
 [3]: http://wiki.nginx.org/WordPress