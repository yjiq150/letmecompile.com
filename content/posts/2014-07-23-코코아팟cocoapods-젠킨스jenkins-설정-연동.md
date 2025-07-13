---
title: 코코아팟(CocoaPods) 젠킨스(Jenkins) 설정 연동
date: 2014-07-23T04:47:28+00:00
url: /코코아팟cocoapods-젠킨스jenkins-설정-연동/
dsq_thread_id:
  - 2865931809
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/use-cocoapod-in-playground/"     class="post-811"><span class="crp_title">Playground에서 Cocoapod 라이브러리 사용하기</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/use-cocoapod-in-playground/"     class="post-811"><span class="crp_title">Playground에서 Cocoapod 라이브러리 사용하기</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
  - 개발도구

---
&nbsp;

코코아팟(CocoaPods)을 사용하게 될경우 프로젝트(.xcproject)기반에서 워크스페이스(.xcworkspace) 기반으로 변경된다. 이때 젠킨스(Jenkins)에서 기존 프로젝트 기반 설정을 그대로 사용할경우 기본 프로젝트는 잘 컴파일되지만, 연결된 Pods 프로젝트가 업데이트 및 컴파일이 되지 않아서 `-lPods` 링크 에러(link error)가 나는데. 이때 Jenkins에서 CocoaPods을 잘 인식할 수 있게 할 수 있는 설정방법을 공유하고자 한다.

## 설정 방법

기존에 프로젝트 기반으로 빌드설정이 되어있던 젠킨스 빌드설정 페이지를 열고,

  1. Build 섹션에서 &#8220;Add build step&#8221; 버튼을 눌러서 Execute shell 항목을 추가 후 `pod install` 명령어 입력</p> 
  2. 새로 추가된 Execute shell 항목은 먼저실행될 수 있도록 Xcode 항목보다 위쪽으로 드래그하여 옮긴다.

  3. Build 섹션에서 Xcode &#8211; Advanced Xcode build options &#8211; Xcode Workspace File 항목에 CocoaPods 의해 자동으로 생성된 워크스페이스 이름을 입력한다(일반적으로 기존 프로젝트명과 동일함).

ex) 워크스페이스 파일명이 `MyExample.xcworkspace` 일 경우 `MyExample` 만 입력한다.

저장하신 후 빌드하면 정상적으로 동작하는것을 확인할 수 있다.