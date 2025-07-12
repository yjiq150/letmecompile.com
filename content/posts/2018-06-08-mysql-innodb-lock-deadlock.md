---
title: 'MySQL InnoDB lock & deadlock 이해하기'
author: yjiq150
type: post
date: 2018-06-07T17:57:28+00:00
url: /mysql-innodb-lock-deadlock/
dsq_thread_id:
  - 6717585445
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/database-transaction-isolation-level/"     class="post-757"><span class="crp_title">DB 트랜잭션 격리 수준</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/database-transaction-isolation-level/"     class="post-757"><span class="crp_title">DB 트랜잭션 격리 수준</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - Database

---
대규모의 많은 요청이 동시에 들어오는 데이터베이스(Database, DB) 어플리케이션의 경우 데이터의 정합성을 유지하면서 최대한 동시성을 높이는 것이 매우 중요한 포인트이다. MySQL InnoDB 엔진의 경우 상황에 따른 여러가지 락(Lock)을 통해서 동시성을 제어하며 이를 통해서 사용자가 설정한 대로 원하는 수준의 [트랜잭션 격리 수준(Transaction Isolation level)][1]을 유지해준다.

MySQL InnoDB 엔진은 사용자가 필요에 따라 명시적으로 locking read를 할 수 있도록 두가지 쿼리를 제공한다.

  * 한 트랜잭션에서 읽어간 데이터를 **다른 트랜잭션에서 배타적으로 수정하기 위해 락을 획득하려 할때 (읽어가는 것은 허용)** 기다리게 만들고 싶다면  
    `SELECT ... LOCK IN SHARE MODE` 사용
  * 한 트랜잭션에서 읽어간 데이터를 **다른 트랜잭션에서 배타적으로 읽거나, 수정하기 위해 락을 획득하려할 때** 기다리게 만들고 싶다면 `SELECT ... FOR UPDATE` 사용 
      * 예) 특정 필드의 값을 읽어온 후 (SELECT), 해당 값을 1 증가시켜서 저장 (UPDATE)하고 싶을때 
          * 이 경우 SELECT .. FOR UPDATE를 사용하지 않으면, 여러 트랜잭션이 동시에 접근할 경우 정상 동작 하지 않을 수 있다. 즉, 트랜잭션 2개가 실행되었는데 값이 2만큼 증가되지 않고 1만 증가되어있을 가능성 존재. 

먼저 이러한 locking 구문들이 실제 어떤 lock들을 통해서 내부적으로 구현되는지 좀더 자세히 알아보도록 하자.

&#42; 참고: MySQL 8.0 부터는 기존 `LOCK IN SHARE MODE` 대신 `FOR SHARE`라고 간략하게 적어줘도 된다. (하위 호환성을 위해 기존 구문도 문제 없이 실행됨)

## MySQL InnoDB 락(Lock)의 종류

실제 Inno DB내부에서는 경우 여러 트랜잭션들이 경합하고 있는 상황에서 최대한의 성능을 위해서 여러 방식의 다양한 락(Lock)을 조합해서 사용하고 있다. 이 과정에서 각 트랜잭션이 획득한 락들은 해당 트랜잭션이 commit/rollback 되어 종료될 때까지 유효하기 때문에, 이러한 lock을 최소화 하면서도 동시성을 유지하기 위해 어떤 상황에 어떤 락들이 실제로 적용되는지를 좀더 자세히 살펴보도록 하자.

### 락의 적용 요소에 따른 분류

  * Shared lock (`S`) 
      * Row-level lock
      * SELECT 위한 read lock
      * shared lock이 걸려있는 동안 다른 트랜잭션이 해당 row에 대해 `X` lock 획득(exclusive write)은 불가능하지만 `S` lock 획득(shared read)은 가능
      * 한 row에 대해 여러 트랜잭션이 동시에 `S` lock을 획득 가능
  * Exclusive lock (`X`) 
      * Row-level lock
      * UPDATE, DELETE 위한 write lock
      * exclusive lock이 걸려있으면 다른 트랜잭션이 해당 row에 대해 `X`, `S` lock을 모두 획득하지 못하고 대기해야 한다.
  * Intention lock 
      * Table-level lock
      * 테이블 안의 &#8220;row에 대해서 나중에 어떤 row-level락을 걸 것&#8221;이라는 의도를 알려주기 위해 미리 table-level에 걸어두는 lock
      * `SELECT ... LOCK IN SHARE MODE` 이 실행되면, 
          * 먼저 intention shared lock (`IS`) 이 테이블에 걸림
          * 그 후 row-level에 `S` lock이 걸림
      * `SELECT ... FOR UPDATE`, `INSERT`, `DELETE`, `UPDATE` 이 실행되면, 
          * 먼저 intention exclusive lock (`IX`) 이 테이블에 걸림
          * 그 후 row-level에 `X` lock이 걸림
      * `IS`, `IX` 락은 여러 트랜잭션에서 동시에 접근이 가능하다 (서로 block하지 않는다). 하지만 동일한 row에 row-level의 실제 lock (`S` 또는 `X`)을 획득하는 과정에서 동시 접근을 막거나 허용하는 제어를 하게된다.</p> 
      * `LOCK TABLES`, `ALTER TABLE`, `DROP TABLE` 이 실행될 때는 `IS`, `IX` 를 모두 block하는 table-level 락이 걸린다. 즉 `IS`, `IX` lock 을 획득 하려는 트랜잭션은 대기상태로 빠짐 
      * 반대로 `IS`, `IX` 락이 이미 걸려있는 테이블에 위의 쿼리를 실행하면 를 실행하면 해당 트랜잭션이 대기상태로 빠짐.
    
      * Table-level에서 한번, row-level에서 한번, 2단계(2-phase)로 lock을 적용하는 이유?
        
          * A 트랜잭션에서 이미 테이블에 대해 락이 걸려있는데, B 트랜잭션에서 해당 테이블의 특정 row에 lock을 거는것을 원천적으로 방지 할 수 있다. (반대의 경우도 마찬가지) 
              * 예) row-level의 write이 일어나고 있을때 테이블 스키마가 변경되서는 안된다. write query의 경우 이미 `IX` 락을 획득한 상태이기 때문에 해당 테이블의 스키마가 변경되는것을 막을 수 있다.

아래는 Share lock, Exclusive lock, Intention lock이 각각 다른 트랜잭션에서 사용될때, 충돌(Conflict, 대기상태 빠짐), 또는 호환(Compatible, 대기상태에 빠지지 않음)이 되는지에 대해 정리된 표이다.

<table>
  <tr>
    <th width="10%">
    </th>
    
    <th width="22.5%">
      <code>X</code>
    </th>
    
    <th width="22.5%">
      <code>IX</code>
    </th>
    
    <th width="22.5%">
      <code>S</code>
    </th>
    
    <th width="22.5%">
      <code>IS</code>
    </th>
  </tr>
  
  <tr>
    <td>
      <code>X</code>
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Conflict
    </td>
  </tr>
  
  <tr>
    <td>
      <code>IX</code>
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Compatible
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Compatible
    </td>
  </tr>
  
  <tr>
    <td>
      <code>S</code>
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Compatible
    </td>
    
    <td>
      Compatible
    </td>
  </tr>
  
  <tr>
    <td>
      <code>IS</code>
    </td>
    
    <td>
      Conflict
    </td>
    
    <td>
      Compatible
    </td>
    
    <td>
      Compatible
    </td>
    
    <td>
      Compatible
    </td>
  </tr>
</table>

### 락이 적용되는 상황에따른 분류

  * Record Lock 
      * primary key 또는 unique index (multi-column unique index 포함)로 조회해서 하나의 인덱스 레코드(=row)에만 lock을 거는 것을 의미
      * 예) `SELECT id FROM t WHERE id = 10 LOCK IN SHARE MODE` 
          * 위 쿼리를 실행하면 id=10 인 레코드에 대해 `S` lock이 걸린다.
      * 예) `SELECT id FROM t WHERE id = 10 FOR UPDATE` 
          * 위 쿼리를 실행하면 id=10 인 레코드에 대해 `X` lock이 걸린다.
      * row-level lock이란 결국 인덱스 레코드에 대해 락을 거는것을 의미한다.
  * Gap Lock (= range lock) 
      * 실제 존재하는 인덱스 레코드에 락을 거는것이 아니고 범위를 지정하기 위해 인덱스 레코드 사이의 범위(gap)에 락을 거는 것을 의미한다.
      * negative infinity (최초 레코드 이전), positive infinity (마지막 레코드 이후) 를 가상의 인덱스 레코드로 생각해서 lock을 적용하는것이 가능하다.
      * Gap lock을 통해서 같은 SELECT 쿼리를 두번 실행했을때 다른 트랜잭션에서 데이터가 수정되었더라도 같은 결과가 리턴되는 것을 보장할 수 있다. (Phantom read 방지) 
          * 예) `SELECT id FROM t WHERE id BETWEEN 10 and 20 FOR UPDATE`
          * 위 쿼리를 실행하면 에서 c1=10~20 사이에 `X`락이 걸리기때문에 다른 트랜잭션에서 c1=15를 가지는 데이터를 INSERT하려면 대기 상태로 빠진다. 
      * &#8220;READ COMMITTED&#8221; isolation level을 사용하면 트랜잭션 내부에서 &#8220;동일 SELECT 실행, 동일 결과&#8221; 보장을 하지 않아도 되기때문에 gap lock 비활성화 해도된다. 이경우 DB의 쿼리 처리량을 늘릴 수 있지만, 그만큼 대기시간이 줄어들고 동시 write가 증가하기때문에 데드락에 걸릴 확률이 높아진다.
      * 컬럼에 대한 WHERE 절로 많은 row들을 제외시키고 하나의 레코드만 추출되었을때, Record lock과 Gap lock 어느것이 사용될까? 
          * 컬럼에 unique index가 걸려있는 경우에는 gap lock이 필요없다. (record lock이 사용됨)
          * 컬럼에 index가 걸려있지 않거나, index가 걸려있어도 unique 하지 않다면 gap lock이 필요하다. 
              * index가 걸려있다면 row를 찾기위해 스캔했던 index range에 대해서 gap lock 적용
              * index가 걸려있지 않다면 결국 테이블 전체를 스캔해야 되기때문에 모든 row에대해 lock이 걸림.
  * Next-Key Lock 
      * 범위를 지정한 쿼리를 실행하게되면 실제로는 위에서 각각 설명했던 record lock (찾아진 인덱스 레코드에 대해)과 gap lock (해당하는 인덱스 레코드 사이사이) 이 복합적으로 사용된다.
      * 다음 그림을 통해 Next-Key lock이 어떤 것인지를 한눈에 살펴보자.

![next_key_lock.png][2] 

  * Insert Intention Lock 
      * INSERT 구문이 실행될 때 InnoDB 엔진 내부적으로 implicit하게 획득하는 특수한 형태의 gap lock. (gap lock은 일반적으로 explicit하게 locking read를 위한 SELECT 구문이 실행될때 발생하지만 이 lock은 INSERT 시점에 implicit하게 자동으로 발생)
      * 여러개의 트랜잭션들이 gap 안의 다른 위치에 INSERT를 동시 수행할 때 기다릴 필요가 없도록 하는것이 목적 (Insert intention lock들 간에는 충돌이 일어나지 않는다)
      * INSERT될 row에 대해서 exclusive lock을 걸기 전에 먼저 insert intention lock을 건다.
      * 예) pk=3, pk=6의 레코드가 존재하는 테이블이 존재 
          * A 트랜잭션에서 pk=5에 INSERT, B 트랜잭션에서 pk=4에 INSERT시도
          * 만약 일반적인 gap lock 사용한다면: 
              * A트랜잭션이 pk=5를 INSERT하는 과정에서 pk=3~5에는 gap lock 걸림
              * B트랜잭션이 pk=4에 INSERT 시도시 pk=3~5에 gap lock이 걸려있기때문에 A가 트랜잭션이 완전히 종료될 때 까지 기다려야 한다.
              * 대기시간 존재!
          * Insert Intention Lock 사용시: 
              * A트랜잭션이 pk=5를 INSERT하는 과정에서 pk=3~5에는 insert intention lock 걸림
              * B트랜잭션이 pk=4에 INSERT 시도시 pk=3~5에 insert intention lock이 걸려있더라도 pk가 겹치지 않기때문에 바로 진행 가능
              * 대기시간 없음!
              * 실제 InnoDB의 동작 방식
  * AUTO-INC Lock 
      * 여러 트랜잭션이 동시에 실행될때 AUTO_INCREMENT 컬럼의 값을 일관성 있게 증가시키기 위해 필요한 lock
      * AUTO_INCREMENT 방식에대해서 InnoDB에서 몇가지 모드를 제공하는데 자세한 내용은 [MySQL &#8211; InnoDB Auto Increment 성능최적화][3] 글을 참고.

## 데이터베이스에서 데드락(Deadlock) 이란?

두개 이상의 트랜잭션들이 동시에 진행될때 서로가 서로에 대한 락을 소유한 상태로 대기 상태로 빠져서 더이상 진행하지 못하는 상황을 데드락(deadlock)이라고 한다.

  * 데드락은 트랜잭션을 지원하는 데이터베이스에서는 자주 발생하는 문제이다.
  * 멀티 스레드(Multi-threaded) 어플리케이션에서 발생하는 데드락은 해당 어플리케이션을 완전히 멈추게 해버리기 때문에 위험하다.
  * 하지만 일반적인 DBMS (Database Management System)에서는 데드락 탐지(Deadlock detection) 기능을 제공하기때문에 데드락이 발견되면 자동으로 해소시켜준다 (실제 데드락 상황이 아닐지라도 락에 대한 대기시간이 설정된 시간을 초과하면 이것도 데드락으로 처리된다)
  * 이 과정에서 작업중이던 트랜잭션들 중 일부가 취소되는 경우가 발생 할 수 있기때문에 어플리케이션 레벨에서 해당 트랜잭션을 재실행 하여 작업을 완수할 수 있도록 구성해야한다.

### 데드락(Deadlock)을 줄일 수 있는 방법

  * 트랜잭션(transaction)을 최대한 간결하게 만든다.
  * 인덱스를 잘 구성해야한다. 더 적게 레코드를 스캔할 수록 더 적은 락이 걸린다.
  * Locking read (`SELECT ... FOR UPDATE`, `SELECT ... LOCK IN SHARE MODE`)를 사용시에 READ_COMMITTED와 같은 더 낮은레벨의 트랜잭션을 사용할 수 있는 상황이라면 적극적으로 사용한다.
  * 트랜잭션 안에서 여러 데이터를 수정할때는 발생하는 Lock의 순서를 항상 순차적으로 만든다. 
      * 즉 A, B, C 테이블을 수정시 각각의 트랜잭션에서 A -> B -> C 순서로 수정하면 데드락의 위험이 적다.
      * 예시) 어플리케이션 상의 특정 조건에따라 A -> C -> B 또는 C &#8211; B -> A 와 같이 각각 트랜잭션이 동시 실행되면서 다른순서로 데이터를 수정하게 된다면 데드락의 확률이 높아진다. 
          * 일반적으로는 row 단위로 lock이 걸리지만 이해를 쉽게 하기 위해 테이블 단위 락을 가정
          * 첫번째 트랜잭션에서 A 테이블 락을 잡음
          * 두번째 트랜잭션에서 C 테이블 락을 잡음
          * 첫번째 트랙잭션에서 C 를 수정하려 하지만 두번째 트랜잭션에서 C 테이블 락을 먼저 잡았기때문에 대기 상태에 빠짐
          * 두번째 트랜잭션이 B 테이블 락을 잡고 수정 후, A 테이블 락을 잡으려 하지만 첫번쨰 트랜잭션이 락을 잡고있기떄문에 대기상태에 빠짐
          * 트랜잭션이 서로 물고물린 상태로 대기상태에 빠짐 (= 데드락)
  * 하나의 구문에 여러 ROW가 포함되는 Batch `INSERT ... ON DUPLICATE KEY UPDATE`를 주의하라. 하나의 Batch 구문은 트랜잭션이 걸린 여러개의 구문 처럼 동작하기때문에 각각의 배치 Query에 포함된 데이터의 PK가 겹치게되면 데드락이 발생할 확률이 있다. 
      * 예시) 아래 Deadlock Case 1 참고
  * 트랜잭션들을 완전히 Serialize 한다 
      * 예시) 
          * 1줄의 데이터만 갖고있는 세마포어용 테이블을 생성
          * 각각의 트랜잭션들이 다른 테이블에 접근하기 전에 먼저 세마포어용 테이블을 업데이트하도록 한다.
          * 세마포어 테이블에 늦게 접근한 트랜잭션은 대기상태에 빠지기 때문에 각 트랜잭션들의 완전한 순차 실행이 보장된다.

### 데드락 상태 확인 명령어

어떤 lock이 걸려있는지 상태를 확인하거나 최근에 발생했던 데드락에 대해 알고 싶으면 아래 명령어를 사용하면 된다.

    SHOW ENGINE INNODB STATUS;
    

아래는 exclusive lock (lock_mode X) 때문에 대기중인 트랜잭션이 있을때의 예제 메시지이다.

    ------- TRX HAS BEEN WAITING 5 SEC FOR THIS LOCK TO BE GRANTED:
    RECORD LOCKS space id 770 page no 3 n bits 80 index PRIMARY of table `example_db`.`test` trx id 774317 lock_mode X locks rec but not gap waiting
    

## 데드락 케이스 분석

### Deadlock Case 1

Upsert 동작(기존 데이터가 존재하면 UPDATE, 없으면 INSERT)의 경우 MySQL에서는 INSERT INTO &#8230; ON DUPLICATE KEY UPDATE 구문을 이용하여 DB차원에서 쿼리로 구현이 가능하기 때문에 매우 편리하다. 하지만 이 여러개의 Row에 대해서 한번에 Batch성으로 실행하게되면 데드락이 발생할 위험이 큰 편이다.

Query A:

     INSERT INTO my_table (pk, name) VALUES(1, 'a'), (2,'b') ON DUPLICATE KEY UPDATE name=VALUES(name);
    

Query B:

     INSERT INTO my_table (pk, name) VALUES(2, 'a'), (1,'b') ON DUPLICATE KEY UPDATE name=VALUES(name);
    

위의 두 쿼리가 DB상에서 다른 connection을 통해 들어와서 동시에 실행되면 각각의 쿼리는 2번의 INSERT를 트랜잭션으로 묶어놓은것 처럼 동작한다. 따라서 운이나쁘게 다음과 같은 순서로 실행되면 데드락이 발생한다.

  * A에서 pk=1인 row upsert pk=1 lock 획득
  * B에서 pk=2인 row upsert pk=2 lock 획득
  * A에서 pk=2인 row upsert 시도 -> B에서 먼저 lock을 가져갔기때문에 대기
  * B에서 pk=1인 row upsert 시도 -> A에서 먼저 lock을 가져갔기때문에 대기
  * 데드락 발생

Upsert가 아닌 WHERE 조건에의해 여러 row들을 한번에 UPDATE 한다면 비슷한 이유로 데드락이 발생할 수 있다.

### Deadlock Case 2

트랜잭션 A, B, C를 가정하고 아래 INSERT 상황과 DELETE 상황에서 어떻게 내부적으로 동작하는지 살펴보자.

#### INSERT 경쟁 상황

  * A, B, C 순서로 같은 key값을 가지는 데이터를 INSERT 하기 위해 경쟁 
      * A 에서 `INSERT INTO table (pk) VALUES (3);` 실행하면 A에서 exclusive lock 획득
      * B와 C에서도 동일한 INSERT 구문 실행한다. (B, C는 대기상태로 빠짐)
      * A를 rollback하는 순간 B, C가 경쟁 시작. 
          * INSERT의 경우 exclusive lock을 획득 시도해야하지만, 해당 INSERT에서 duplicated key error가 발생하는 경우에는 해당 인덱스 레코드에 대해 일단 shared lock을 먼저 획득 시도하는 특성이 있다.
          * B, C가 동일 인덱스 레코드에 대해 shared lock을 먼저 획득한 후 exclusive lock을 잡으려고 하기 때문에 데드락이 발생. (B가 exclusive lock을 획득하려 해도 C가 획득한 shared lock에 의해 불가능, C -> B의 경우에도 vice versa)
          * 늦게 실행된 C가 deadlock처리되어 트랜잭션이 롤백되어 종료되면, B가 exclusive lock을 획득하여 실행된다.
      * A를 commit하면 B, C는 바로 duplicated key error가 발생하지만 여전히 트랜잭션은 열려있다.

결국 이런 INSERT 상황에서 선행 트랜잭션이 rollback하게되면 나머지 트랜잭션들 중 하나만 성공하고 나머지는 모두 데드락으로 강제 롤백 된다는 것을 알 수 있다. 이 케이스는 얼핏 봐서는 데드락이 발생하지 않을 것 같은 상황처럼 보이기 때문에 문제 발견이 쉽지 않으므로 기억해두는 것이 좋다.

#### DELETE 경쟁 상황

INSERT의 경우가 매우 특이한 경우라는 것을 강조하기위해서 DELETE에 대해서는 특이하지 않게 잘 동작하는 것을 확인해보자.

  * A, B, C 순서로 같은 key값에 대한 DELETE로 경쟁시키는 경우 
      * A가 exclusive lock 획득
      * B가 exclusive lock 획득시도(대기), C가 exclusive lock 획득시도(대기)
      * A가 commit 하는 경우 
          * lock의 대상이었던 row가 사라짐
          * 때문에 B, C 모두 대기상태가 종료되면서 affected row=0 결과 리턴함 (B, C 트랜잭션은 계속 열려있음)
      * A가 rollback 하는 경우 
          * B가 exclusive lock 획득 C는 계속 대기

### INSERT, DELETE 상황 비교 분석

  * INSERT의 경우 duplicated key error 가 발생시 shared lock을 획득 시도하기 때문에 하나 이상의 트랜잭션이 동시에 동일한 row에 대한 shared lock을 획득하게 된다. (shared lock은 여러명이 동시에 획득 가능하기 때문) 트랜잭션 A가 종료되면, 다시 트랜잭션 B가 exclusive lock을 획득 시도하려 하지만 트랜잭션C가 잡고있는 shared lock 때문에 exclusive lock을 획득하는것이 불가능하다. 반대의 경우도 마찬가지이므로 데드락 상황이다.
  * DELETE의 경우 exclusive lock을 획득 시도하기 때문에 한번에 하나의 트랜잭션만 lock을 획득가능하고 나머지는 대기한다. 따라서 데드락 상황이 생기지 않는다. 

## 참고 자료

https://dev.mysql.com/doc/refman/5.7/en/innodb-locks-set.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-deadlocks-handling.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-next-key-locking.html

https://stackoverflow.com/questions/25903764/why-is-an-ix-lock-compatible-with-another-ix-lock-in-innodb



<div style="margin-bottom:5px">
  <strong> <a href="https://www.slideshare.net/billkarwin/innodb-locking-explained-with-stick-figures" title="InnoDB Locking Explained with Stick Figures" target="_blank">InnoDB Locking Explained with Stick Figures</a> </strong> from <strong><a href="https://www.slideshare.net/billkarwin" target="_blank">Karwin Software Solutions LLC</a></strong>
</div>

https://github.com/octachrome/innodb-locks

 [1]: https://www.letmecompile.com/database-transaction-isolation-level/
 [2]: /uploads/2018/06/next_key_lock.png
 [3]: https://www.letmecompile.com/mysql-innodb-auto-increment-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94/