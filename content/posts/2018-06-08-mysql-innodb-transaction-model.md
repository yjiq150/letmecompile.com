---
title: MySQL InnoDB Transaction Model 이해하기
date: 2018-06-07T18:03:48+00:00
url: /mysql-innodb-transaction-model/
dsq_thread_id:
  - 6720180595
dsq_needs_sync:
  - 1
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/database-transaction-isolation-level/"     class="post-757"><span class="crp_title">DB 트랜잭션 격리 수준</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/database-transaction-isolation-level/"     class="post-757"><span class="crp_title">DB 트랜잭션 격리 수준</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - Database

---
MySQL의 InnoDB 엔진은 [SQL 표준에 정의된 4가지 트랜잭션 격리 수준(transaction isolation level)][1]을 모두 제공한다. InnoDB 엔진의 트랜잭션 격리 수준 기본값은 REPEATABLE READ이다. MySQL client는 `SET TRANSACTION` 구문을 실행해서 격리 수준을 변경할 수 있고, server 자체에 옵션을 줘서 모든 client 연결에 대해 동일한 격리 수준을 적용하는 방법도 있다.

그러면 각각의 트랜잭션 격리 수준을 제공하기위해서 MySQL의 InnoDB 엔진이 어떤 방식을 사용하고있는지 가장 많이 사용되는 격리 수준 순으로 각각 알아보도록 하자.

## 1. REPEATABLE READ

앞서 언급했듯이 REPEATABLE READ 레벨의 경우 InnoDB 엔진에서 사용하는 디폴트 트랜잭션 격리 수준이고 특별한 성능 이슈가 없다면 잘 변경하지 않기때문에 가장 많이 사용된다. REPEATABLE READ를 만족시키기 위해서 MySQL InnoDB 엔진은 아래 두가지 방식을 사용한다.

  1. locking을 이용하지 않는 consistent read 방식을 사용 (자세한 내용은 다음 섹션을 참고)
  2. 명시적으로 locking read를 할 수 있도록 `SELECT ... FOR SHARE`, `SELECT ... FOR UPDATE` 구문을 제공

### REPEATABLE READ에서 locking이 어떻게 사용되는지 케이스별 분석

locking read 구문들과, UPDATE, DELETE이 수행될 때 실제 lock이 적용되는 방식은 SQL statement의 조건과, 대상 테이블의 컬럼에 index가 걸려있는지, 해당 index가 unique 한지에 따라 달라진다. 아래에 상황별로 정리를 해 보았다.

  * unique index가 적용된 컬럼을 특정 값으로 검색 
      * 예) `... WHERE pk=8`
      * 찾아진 하나의 인덱스 레코드에만 lock적용
  * 그 외의 경우 
      * unique index 컬럼을 범위로 검색 
          * 예) `... WHERE pk > 100` 
      * non-unique index 컬럼을 특정 값 또는 범위로으로 검색 
          * 예) `... WHERE field = 3` or `... WHERE field > 4`
      * gap lock 또는 next-key lock을 이용해서 스캔한 인덱스 범위에 lock을 적용해서 다른 세션이 해당 범위에 INSERT하는 것을 막는다.

위에 언급된 개별적인 lock 들에 대한 더 자세한 설명은 [MySQL InnoDB lock & deadlock][2] 포스팅을 참고하면 된다.

### Consistent read

  * 트랜잭션 내부에서 non-locking read(기본 SELECT 구문) 실행할 때, 동시에 실행중인 다른 트랜잭션에서 데이터를 변경하더라도 특정 시점의 스냅샷(snapshot)을 이용하여 기존과 동일한 결과를 리턴할 수 있도록 해주는 기능. 
      * Consistent nonlocking read 라고도 불리는데 consistent read와 동일하다.
      * MVCC (Multi Version Concurrency Control)의 한 방식으로 오라클 DB에서는 consistent snapshot이라는 용어로 사용된다.
      * Lock을 사용하지 않고 스냅샷을 기반으로 동작하기 때문에 동시성이 높다.
  * 설정된 트랜잭션 격리 수준에 따른 스냅샷이 생성되는 시점 비교 
      * REPEATABLE READ인 경우 트랜잭션 시작 후 첫번째 read operation이 수행되는 시점의 데이터로 스냅샷이 생성된다. 
          * [SQL 표준 트랜잭션 격리 수준][1]에 따르면 REPEATABLE READ의 경우 phantom read가 발생한다.
          * 하지만 MySQL InnoDB 엔진의 경우 consistent read를 사용하기 때문에 **REPEATABLE READ 레벨임에도 불구하고 phantom read가 발생하지 않는다.**
      * READ COMMITTED인 경우 매 read operation이 발생하는 시점의 데이터로 스냅샷이 다시 생성된다. 

### Consistent read의 문제점

  * Consistent read 제약조건 
      * 트랜잭션 내에서 SELECT를 실행하면 다른 트랜잭션에서 데이터를 변경했더라도 항상 동일한 결과를 리턴한다.
  * 트랜잭션 제약조건 
      * 현재 트랜잭션 내에서 수행한 변경사항(INSERT/DELETE/UPDATE)에 대한 결과가 트랜잭션 내에서는 항상 보여야 한다.
  * 위 두가지 조건이 결합하면 문제 상황이 발생한다. 
      * 트랜잭션 내에서 특정 row를 UPDATE한 후 해당 row에 대해 SELECT문을 실행하면 최신 데이터를 불러온다. (UPDATE된 row를 제외한 나머지 row는 스냅샷에서 불러온다.)
      * 해당 row에 대해서만 최신 데이터를 불러오다보니 이 데이터는 다른 트랜잭션에서 변경한 사항이 포함된 데이터일 가능성이 있다. 이로인해 REPEATABLE READ가 깨진다.
      * 예) 데이터가 비어있는 my_table을 가정 
          * Tx1, Tx2 시작
          * Tx2에서 SELECT -> return 0 rows (첫 SELECT에 스냅샷 생성) 
          * Tx1에서 `INSERT INTO my_table (pk, value) VALUES (1, 'a')` 실행 후 commit (Tx1 종료)
          * (Tx2는 계속 진행중)
          * Tx2에서 SELECT -> return 0 rows (REPEATABLE READ 유지 중)
          * Tx2에서 UPDATE my_table SET value=&#8217;b&#8217; WHERE pk=1 -> &#8216;1&#8217; row affected
          * Tx2에서 SELECT -> return 1 rows (REPEATABLE READ 깨짐!)
  * 추가로 consistent read는 DDL (Data Description Language) 구문에 대해서는 동작하지 않는다는 사실에 주의해야 한다. 
      * `DROP TABLE`, `ALTER TABLE`이 실행되고 난 후에는 스냅샷이 더이상 유효하지 않다.

## 2. READ COMMITTED

  * InnoDB 엔진은 READ COMMITTED 레벨에서도 앞서 설명한 consistent read를 사용한다. 하지만 REPEATABLE READ일 때와는 다르게 **매번 read opration 작업이 있을 때 마다 새로운 스냅샷을 생성**해서 읽어오기 때문에 Phantom read가 발생할 수 있다.</p> 
  * REPEATABLE READ일 때 처럼 locking read를 할 수 있도록 동일하게 `SELECT ... FOR SHARE`, `SELECT ... FOR UPDATE` 구문이 제공되지만 **적용되는 lock의 범위가 달라지는 것에 주의**해야 한다.

### READ COMMITTED 레벨에서 locking이 어떻게 사용되는지 케이스별 분석

READ COMMITTED 레벨로 설정된 경우 동일하게 locking read 구문 및, UPDATE, DELETE이 수행되더라도 REPEATABLE READ일 때보다 더 적은 범위에 대해서 lock이 적용된다.

  * locking read, UPDATE, DELETE 구문이 실행될때 &#8220;찾아진 레코드&#8221;에만 락을 건다.
  * 레코드를 찾기위해 스캔했던 인덱스 레코드에 대해서는 gap lock을 적용하지 않기 때문에 해당 gap에 대해 다른 트랜잭션에서 자유롭게 INSERT가 가능하다. (Phantom read 발생)
  * foreign-key 제약과 duplicate-key 확인을 위해서만 gap lock이 사용된다.
  * lock이 적어지는 만큼 동시성이 좋아진다.
  * deadlock이 발생할 확률이 REPEATABLE READ보다는 줄어들지만, 여전히 발생 가능성은 존재한다.

## 3. READ UNCOMMITTED

READ UNCOMMITTED 레벨의 경우 lock을 사용하지 않는다. 이 레벨의 경우 트랜잭션을 사용하는 의미 자체가 거의 없으므로 잘 사용되지 않는다.

## 4. SERIALIZABLE

트랜잭션 격리 수준이 SERIALIZABLE로 설정되면 InnoDB는 자동으로 일반적인 SELECT 구문을 `SELECT ... FOR SHARE`으로 변경하여 실행한다. 그 외에는 REPEATABLE READ 레벨과 동일하다.

## 참고

https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html

https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html

 [1]: https://www.letmecompile.com/database-transaction-isolation-level/
 [2]: https://www.letmecompile.com/mysql-innodb-lock-deadlock/