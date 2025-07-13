---
title: Cocoapod version update for Mac OS Yosemite
date: 2014-10-27T15:31:08+00:00
url: /cocoapod-version-update-for-mac-os-yosemite/
dsq_thread_id:
  - 3167780277
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/use-cocoapod-in-playground/"     class="post-811"><span class="crp_title">Playground에서 Cocoapod 라이브러리 사용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/use-cocoapod-in-playground/"     class="post-811"><span class="crp_title">Playground에서 Cocoapod 라이브러리 사용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - 요세미티
  - 코코아팟
  - 업데이트

---
   
Mac OS X Yosemite 로 업그레이드를 하게되면 cocoaPod이 잘 작동하지 않을것이다. 이때 다음 인스트럭션을 따라 해결해 나가면 된다.

  1. 코코아팟 최신버전으로 업데이트 (현재 최신은 0.34.4 입니다) 
        sudo gem uninstall cocoapods
        sudo gem uninstall xcodeproj
        sudo gem install xcodeproj
        sudo gem install cocoapods
        

   
2. Podfile의 첫줄에 다음 코드 추가

        source 'https://github.com/CocoaPods/Specs.git'
    

  * 이부분은 옵셔널하긴한데, 추가하지 않으면 deprecated warning이 뜨기때문에 추가해주는것이 좋다.

   
3. 빌드시 Podfile.lock 혹은 Manifest.lock 관련 에러가 날 경우 처리방법

  * 코코아팟이 버전이 올라가면서 *.xcconfig파일들이 configuration 별로 생성되는 형태로 바뀌어서 발생하는 에러이다.
  * Xcode의 프로젝트 설정 -> Configurations ->  Debug, Release, inhouse에 각각 Pods.debug, Pods.release, Pods.inhouse 로 컨피그 파일을 지정해주면 해결된다.