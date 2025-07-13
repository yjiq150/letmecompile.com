---
title: 개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac
date: 2019-04-05T15:37:58+00:00
url: /intellij-shortcut-keys-mac/
dsq_thread_id:
  - 7339437854
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/xcode-%eb%8b%a8%ec%b6%95%ed%82%a4-%eb%aa%a8%ec%9d%8c/"     class="post-860"><span class="crp_title">자주쓰는 Xcode 단축키 모음</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/xcode-%eb%8b%a8%ec%b6%95%ed%82%a4-%eb%aa%a8%ec%9d%8c/"     class="post-860"><span class="crp_title">자주쓰는 Xcode 단축키 모음</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 개발도구
tags:
  - 인텔리제이
  - intelliJ

---
IntelliJ를 사용하는 개발자라면 알고있어야 할 실무에서 많이 사용되는 필수적인 단축키들을 정리해보았다.  
여기 나오는 단축키들만 마스터해도 생산성이 팍팍 올라가고 어디가서 IntelliJ 좀 쓸 줄 안다고 어깨에 힘줘 볼 수 있을것이다.  
추가로 단축키 외에도 아래 내용 중에 소개하는 scratch 기능을 이용하면 간단한 코드들을 빠르게 작성하고 테스트 볼 수 있어서 생산성을 많이 올릴 수 있으니 사용해 보는것을 강력히 추천한다.

  * CMD + N : 제네레이트 (컨스트럭터, 게터 셋터)
  * CMD + SHIFT + N (scratch 파일 생성) 
      * 사용예: 
          * 코드 선택된 상태로 누르면 해당 영역이 그대로 복사됨
          * 간단하게 언어 특성이나 코드 실행결과를 빠르게 테스트 보는데 좋음
      * 사용예: 
          * json 생성 후 CMD + OPT + L 하면 auto formating
  * 실행/디버깅 
      * CTRL + D : 디버깅
      * CTRL + OPT + D : 여러 configuration들이 존재할때 특정 configuration 선택할 수 있는 창을 띄움
      * CTRL + SHIFT + D : 현재 커서가 있는 파일 또는 유닛테스트를 build & debug
      * 참고: 위 세가지 단축키의 D 대신 R을 입력하면 디버깅 대신 실행 모드로 동작
      * F8: 디버깅 중에 누르면 next line으로 진행
      * CMD + OPT + R : 현재 브레이크포인트에 멈춰있는 어플리케이션을 Resume
      * CMD + F8 : 현재 커서에 브레이크 포인트 토글
  * CMG + , : InterlliJ 전체 설정
  * CMD + ; : 프로젝트 설정
  * OPT + ENTER : 밑줄친 곳에서 추가액션 
      * lint 적용, 오타 보정, error correction 등등 다양한 액션 가능
  * 아이템 찾기 
      * CMD + (SHIFT or OPT) + O : find symbols, files
      * CMD + SHIFT + a : find actions (인텔리제이의 수많은 메뉴와 기능들을 찾기 힘들때는 이곳을 통해서 검색할것)
      * SHIFT 2회 : find all
      * CMD + E : 최근 열었던 파일 목록
  * 텍스트 찾기 
      * CMD + SHIFT + F 
          * text 전체 찾기
          * scope 지정 가능
      * CMD + SHIFT + R 
          * text 전체 Replace
  * CMD + F12 : current file&#8217;s structure
  * 코드 찾기 
      * OPT + F7 : find usage
      * CTRL + OPT + H: call hierachy
      * CTRL + H: type hierachy
      * CMD + Click: Jump to definition
      * CMD + OPT + Click: Jump to Implementation 
          * interface를 구현한 구현체들을 검색해서 이동
  * CMD + DELETE : 현재 커서가있는 한줄 삭제
  * OPT + F1 → 1 : 현재파일 프로젝트 트리에서 열기
  * SHIFT + F6 : 이름 바꾸기 (refactor)
  * CMD + SHIFT + V : 클립보드 히스토리 보기
  * 네비게이션 
      * CMD + [ : 뒤로 이동
      * CMD + ] : 앞으로 이동
  * 윈도우 분할해서 사용중일때 
      * (단축키 없음) Move to Opposite Group 현재 열려있는 파일을 반대편 윈도우로 옮긴다.