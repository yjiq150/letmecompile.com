---
title: Cocoa Binding Controller Keys
date: 2014-06-14T07:37:57+00:00
url: /cocoa-binding-controller-keys/
dsq_thread_id:
  - 2792037638
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - iOS
tags:
  - ios 바인딩
  - 맥 바인딩

---
코코아 바인딩 컨트롤러의 경우 최소한의 코딩만으로 model값이 변화할때 view와 model 사이의 싱크를 맞춰주는 역할을 한다.

애플에서 제공하는 각 클래스별 바인딩 가능한 키값들은 [다음 문서][1]  
에서 확인하면 되고,

본 글에서는 자주 사용하는 바인딩 컨트롤러들의 상속구조를 살펴보고, 각각의 컨트롤러들에 어떤 키/경로 값을 설정할 수 있는지 정리해 보았다.

## 바인딩 컨트롤러 상속구조

  * NSController -> NSObjectController
  * NSController -> NSObjectController -> NSArrayController
  * NSController -> NSObjectController -> NSArrayController -> NSDictionaryController
  * NSController -> NSObjectController -> NSTreeController
  * NSController -> NSUserDefaultsController

## NSObjectController

  * canAdd
  * canRemove
  * isEditable
  * selectedObjects
  * selection

## NSArrayController

  * arrangedObjects
  * canAdd
  * canInsert
  * canRemove
  * canSelectNext
  * canSelectPrevious
  * filterPredicate
  * isEditable
  * selectedObjects
  * selection
  * selectionIndex
  * selectionIndexes
  * sortDescriptors

## NSDictionaryController

  * arrangedObjects
  * canAdd
  * canInsert
  * canRemove
  * canSelectNext
  * canSelectPrevious
  * isEditable
  * selectedObjects
  * selection
  * selectionIndex
  * selectionIndexes
  * sortDescriptors

## NSTreeController

  * arrangedObjects
  * canAdd
  * canAddChild
  * canInsert
  * canInsertChild
  * canRemove
  * isEditable
  * selectedObjects
  * selectedNodes
  * selection
  * selectionIndexPath
  * selectionIndexPaths
  * sortDescriptors

## NSUserDefaultsController

  * hasUnappliedChanges
  * values

 [1]: https://developer.apple.com/library/mac/documentation/cocoa/reference/CocoaBindingsRef/CocoaBindingsRef.html#//apple_ref/doc/uid/10000189-BCICJDHA