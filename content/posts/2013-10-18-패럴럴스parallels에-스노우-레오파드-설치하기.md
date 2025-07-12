---
title: 패럴럴스(Parallels)에 스노우 레오파드 설치하기
author: yjiq150
type: post
date: 2013-10-18T04:46:38+00:00
url: /패럴럴스parallels에-스노우-레오파드-설치하기/
dsq_thread_id:
  - 1870403762
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 맥 Tips
tags:
  - 스노우레오파드
  - 가상머신
  - 패럴렐스
  - 스노우 레오파드 서버

---
맥 어플리케이션을 개발할 때 [하위 호환성(backword compatibility)을 유지하기 위해 신경써야 할 점][1]들에 관하여 예전에 포스팅을 했었다. 개발할 때 그런 점들을 신경쓰더라도 실수로 놓치는 부분이 많기때문에 실제로 실행되는지 해당 버전의 OS에서 실행하는 절차는 필수이다. 따라서 10.6 Snow Leopard(스노우레오파드) 까지 지원하려면 가상머신에 스노우레오파드를 설치하여 테스트에 사용하는 것이 가장 편리하다.

하지만 라이센스 문제때문인지, 패럴럴스에 가상머신형태 Mac OS를 설치할 경우 Snow Leopard Server만 설치가능하도록 되어있다. 이부분을 우회하기위해 일반적인 Snow Leopard 설치 시디를 갖고 Snow Leopard Server인것처럼 만들어서 패럴럴스를 속여서 설치하는 방법을 소개한다.

## 설치 순서

  1. 디스크 유틸리티 등을 이용하여 갖고있는 스노우 레오파드 설치 시디를 쓰기가능한 dmg 혹은 iso 형태로 생성한다.

  2. 패럴럴스가 스노우 레오파드 서버를 설치하는 것처럼 생각하게 하기위해 1번 과정에서 생성한 스노우레오파드의 이미지 파일을 마운트 한 후 `SystemVersion.plist`를 수정해야 한다. 터미널을 이용하여 다음 명령어를 실행하면 된다.
    
    > touch /Volumes/디스크이미지명/System/Library/CoreServices/SystemVersion.plist

  3. 이제 패럴럴스에서 1-2번과정을 통해 만들어진 이미지를 불러와서 설치를 시작하자. 아래 스크린샷에 해당하는 화면이 나왔을때 상단 메뉴에서 `Utilities` &#8211; `Terminal`을 실행하여 다음 명령어들을 실행한다.
    
    [<img loading="lazy" src="/uploads/2013/10/snowleopard.png" alt="snowleopard" width="1030" height="823" class="alignnone size-full wp-image-209" />][2]
    
    > cd /Volumes/Macintosh\ HD/  
    > mkdir -p System/Library/CoreServices/  
    > touch System/Library/CoreServices/ServerVersion.plist
    
    이 과정을 통해서 설치되고 난 후 가상머신 내부 디스크에 있는 정보파일까지 Snow Leopard Server인것 처럼 속일 수 있게된다.
    
    > 주의: 차후 OS업데이트를 할경우, `ServerVersion.plist`를 업데이트 전 백업한 후, 인스톨이 시작되는 순간 다시 돌려놓는 방식으로 해야 유지가 된다. 그렇지 않으면 업데이트가 끝난 후 재부팅시 Snow Leopard Server로 인식이 되지 않아 부팅 할수 없는 경우가 발생한다. 따라서 업데이트시 항상 만일에 대비하여 스냅샷을 찍어 놓는것을 추천한다.

  4. 터미널을 종료하고 계속 설치를 진행하면 스노우레오파드가 가상머신에서 잘 실행되는것을 볼 수 있다.

참고: 가상머신이 아닌 아닌 일반 PC에 Mac OS 설치를 원한다면 다음 사이트를 방문해 보는 것을 추천한다. http://www.leohazard.com/

 [1]: http://www.letmecompile.com/ios-mac-os-%EC%95%B1-%EA%B0%9C%EB%B0%9C%EC%8B%9C-%ED%95%98%EC%9C%84-%ED%98%B8%ED%99%98%EC%84%B1-%EC%9C%A0%EC%A7%80
 [2]: /uploads/2013/10/snowleopard.png