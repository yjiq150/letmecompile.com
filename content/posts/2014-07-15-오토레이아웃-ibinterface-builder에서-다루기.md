---
title: 오토레이아웃 IB(Interface Builder)에서 다루기
date: 2014-07-15T13:15:37+00:00
url: /오토레이아웃-ibinterface-builder에서-다루기/
dsq_thread_id:
  - 2847818448
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li><li><a href="https://www.letmecompile.com/xcode-%eb%8b%a8%ec%b6%95%ed%82%a4-%eb%aa%a8%ec%9d%8c/"     class="post-860"><span class="crp_title">자주쓰는 Xcode 단축키 모음</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/intellij-shortcut-keys-mac/"     class="post-854"><span class="crp_title">개발자라면 알아야 할 IntelliJ 필수 단축키 20선 for Mac</span></a></li><li><a href="https://www.letmecompile.com/xcode-%eb%8b%a8%ec%b6%95%ed%82%a4-%eb%aa%a8%ec%9d%8c/"     class="post-860"><span class="crp_title">자주쓰는 Xcode 단축키 모음</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS

---
## 제약조건(constraint) 설정

  * 하나의 뷰를 선택 &#8211; 해당 뷰와 슈퍼뷰와의 관계 설정가능, 선택된 뷰 자체의 제약사항 설정가능
  * 두개 이상의 뷰를 선택:  
     선택된 뷰들간의 관계 설정가능, 선택된 뷰들의 공통제약사항 설정가능(ex: equal width)
  * IB왼쪽의 ViewTree화면이나 IB 화면에서 에서 하나의 뷰를 선택 후 컨트롤 키를 누르고 드래그하여 다른 아이템을 선택하면 두 아이템 관계를 설정 가능.

IB에서 최상위 view를 선택한 후, 오른쪽 속성창에서 orientation, top bar, bottom bar 유무 등 옵션을 조정하여 실제보이는 형태를 실행하지 않고 시뮬레이트 가능.

## 레이아웃 <-> 제약조건 서로 반영하기

  * 제약조건이 설정된 상태에서, IB에서 뷰를 드래그해서 변경을해도 제약조건은 그대로이다. 따라서  드래그해서 변경된 뷰 상태를 반영하는 새 제약조건을 적용하고 싶은 경우 Update Constraint (cmd + shift + = ) 동작을 수행해야한다.

  * 반대로 제약조건을 변경한 후, IB에서 실제 뷰들이 어떻게 위치되는지 보고싶은경우, Update Frame( cmd + alt + = ) 동작을 수행할것.

## 오토레이아웃 가이드 색상의 의미

  * 파란색 &#8211; 정상
  * 노란색 &#8211; 실제 설정된 제약조건과 IB상에 보이는 값의 차이를 표시, 파란색으로 돌리려면 업데이트 필요
  * 빨간색 &#8211; 제약조건 충돌. (ex: 가로길이고정인 상태에서  좌/우 마진이 다들어간 경우 등 과도한 제약조건이 설정된경우)