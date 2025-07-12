---
title: DB 트랜잭션 격리 수준
author: yjiq150
type: post
date: 2018-06-04T12:12:28+00:00
url: /database-transaction-isolation-level/
dsq_thread_id:
  - 6710862100
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Database

---
Thread에서 공유 자원에 동시 접근을 제한하기 위해 Lock을 걸듯이 DB에서도 트랜잭션들 간에 같은 동일한 데이터에 대한 동시 접근을 제한하기 위하여 Lock을 설정할 수 있다. Lock을 건다는 것은 그만큼 동시처리량이 줄어든다는 의미이기 때문에 과도하게 사용하면 성능에 문제가 생길 수 있다. 따라서 상황별로 꼭 필요한 수준의 Lock을 걸어서 정합성을 유지하고 최대한의 성능을 내기 위해서는 동시성 이슈가 발생할 수 있는 상황들을 정확히 정의하여 구분하는 것이 매우 중요하다.

다음에 설명하고있는 트랜잭션 격리 수준(Transaction isolation level)을 통해 이 부분을 명확히 정의 할 수 있다.

## DB 트랜잭션 격리 수준(Transaction isolation level)에 따른 동시성 이슈

가장 낮은 isolation 레벨 0의 경우 lock이 걸리지 않기때문에 속도는 매우 빠르나 동시 접근을 허용하기때문에 데이터 정합성에 문제가 생길 수 있고, 반대로 가장 높은 레벨3의 경우 완전히 lock을 걸어 동시 접근을 차단하고 순차적으로 처리 (serializable) 하기 때문에 정합성은 완벽하지만 동시에 처리 할 수 있는 양이 적어 속도가 매우 느리다.

아래 표를 보면서 각각의 transaction isolation level에서 어떤 동시성 이슈들이 발생할 수 있는지 동시성 이슈에 대한 설명과 함께 읽어보면 쉽게 이해할 수 있다.

| Isolation Level     | Dirty Read | Nonrepeatable Read | Phantom Read |
| ------------------- |:----------:| ------------------:| ------------:|
| 레벨0 Read Uncommited |     발생     |                 발생 |           발생 |
| 레벨1 Read Committed  |     X      |                 발생 |           발생 |
| 레벨2 Repeatable Read |     X      |                  X |           발생 |
| 레벨3 Serializable    |     X      |                  X |            X |

앞서 여러번 말했듯이 동시성 이슈란 **하나 이상의 트랜잭션이 동시에 동일한 데이터에 접근하여 읽기/쓰기 오퍼레이션을 할 때 발생** 할 수 있으므로 이 상황을 머리 속에 넣어두고 어떤 문제가 발생할지 상상한 후 다음 이슈들을 살펴보자.

#### Dirty Read

  * 트랜잭션1에서 A테이블을 SELECT 한 후 트랜잭션2에서 A테이블 내용을 변경하는 상황 가정
  * 트랜잭션2가 해당 변경사항을 commit 하지도 않았는데, 트랜잭션1에서 다시 A테이블을 SELECT하면 해당 변경사항을 읽어들일 수 있게됨

#### Non-Repeatable Read (Inconsistent Read)

  * 트랜잭션1에서 A테이블을 SELECT 한 후 트랜잭션2에서 A테이블 내용을 변경(UPDATE)하는 상황 가정
  * 트랜잭션2가 해당 변경사항(UPDATE)을 commit 한 이후에, 트랜잭션1에서 다시 A테이블을 SELECT하면 해당 변경사항을 읽어들일 수 있게됨

#### Phantom Read

  * 트랜잭션1이 A테이블에서 SELECT한 이후 트랜잭션2에서 A테이블에 내용을 추가/삭제(INSERT/UPDATE)하는 상황 가정
  * Repeatable Read가 보장된 경우, A테이블에서 SELECT해왔던 데이터들을 다른 트랜잭션2가 수정(UPDATE)하여 commit한 후 트랜잭션1에서 다시 A테이블을 SELECT 하더라도 트랜잭션2의 수정내용을 읽어들일 수 없다.
  * 하지만 트랜잭션2가 추가/삭제(INSERT/DELETE)를 한 경우, 다시 A 테이블에서 SELECT하게되면 기존에 A에서 SELECT했던 데이터에서 row가 추가되거나 사라질 수 있다.(유령 데이터)

## 참고 사항

위에 명시된 동시성 이슈가 발생하지 않는다고해서 해당 트랜잭션 격리 수준을 만족시키는 것은 아니다. 반대로 동시성 이슈가 발생하면 해당 트랜잭션 격리수준을 만족 시키지 못한다.  
&#8211; 예) Repeatable Read 레벨에서 Phantom Read가 발생하지 않는다고 무조건 Serializable 레벨이 될 수 있는 것은 아니다.  
&#8211; 예) Phantom Read가 발생한다면, 해당 트랜잭션 격리 수준은 Serializable 레벨이 될 수 없다.