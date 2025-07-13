---
title: Playground에서 Cocoapod 라이브러리 사용하기
date: 2018-11-24T18:17:03+00:00
url: /use-cocoapod-in-playground/
dsq_thread_id:
  - 7069216371
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
엑스코드 플레이그라운드(Xcode Playground)에서 간단하게 코드를 테스트 해보고 싶은데 해당 코드가 특정 cocoapod 라이브러리에 의존성이 있는 경우 `cocoapods-playgrounds` 명령어 도구를 사용하면 편리하다.

## 설치 및 사용 방법

### 설치

    sudo gem install cocoapods-playgrounds
    

### 플레이그라운드 프로젝트 생성

    cocoapods playgrounds podName
    

위 명령어를 실행하면 위 라이브러리가 연결된 워크스페이스가 자동으로 생성되고, 해당 워크스페이스 안에 우리가 사용할 수 있는 playground 파일까지 포함되어있으니 여기서 마음껏 테스트 해보면 된다.

### 여러 라이브러리 동시 참조

여러개 라이브러리에 의존성을 가지는 플레이그라운드 프로젝트를 만들고 싶을 경우, 라이브러리 이름을 컴마로 구분하여 붙여 써 둔다.

    pod playgrounds ReactiveKit,ReactiveReSwift,Bond
    

## 트러블 슈팅

`gem install cocoapods-playgrounds`를 통해 설치한 후 명령어를 실행시 Xcode 10에서 아래와 같은 에러 발생한다.

> Errno::ENOENT &#8211; No such file or directory @ dir_initialize &#8211; /Applications/Xcode.app/Contents/Developer/Library/Xcode/Templates/File Templates/Source/Playground with Platform Choice.xctemplate 

이 경우 다음 명령어를 통해서 직접 소스코드로 부터 gem을 빌드한 후 직접 설치하면 문제 해결됨. 현재 최신버전은 1.2.2으로 되어있는데, 버전이 업데이트 되는경우 이에 맞게 아래 명령어를 적절히 수정해 서 실행 할 것.

    git clone https://github.com/asmallteapot/cocoapods-playgrounds.git
    gem build cocoapods-playgrounds.gemspec
    sudo gem install cocoapods-playgrounds-1.2.2.gem