---
title: MySQL – InnoDB Auto Increment 성능 최적화
author: yjiq150
type: post
date: 2018-05-22T19:06:07+00:00
url: /mysql-innodb-auto-increment-성능-최적화/
dsq_thread_id:
  - 6685674989
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/database-transaction-isolation-level/"     class="post-757"><span class="crp_title">DB 트랜잭션 격리 수준</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/database-transaction-isolation-level/"     class="post-757"><span class="crp_title">DB 트랜잭션 격리 수준</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Database

---
MySQL에서 벌크 인서트를 실행할때 내부 동작을 찾아보다 보니, 어찌어찌 MySQL InnoDB 스토리지 엔진 내부에서 AUTO_INCREMENT(오토인크리먼트)를 어떤식으로 핸들링하고 있는지가 더 궁금해져서 MySQL 레퍼런스 문서(MySQL 5.7 기준)를 읽으면서 아래 내용을 요약해보았다.

이제까지 AUTO_INCREMENT 컬럼에 대해서 아무 고민없이 값이 자동으로 잘 증가하겠거니 하고 사용해 왔을텐데, 내부적인 동작 방식을 정확히 이해해서 concurrency를 높여서 성능을 향상시킬 수 있는 튜닝 포인트를 확인해 보자.

### 인서트의 종류 구분

설명에 들어가기에 앞서 다양한 상황을 좀더 쉽게 설명하기 위해 인서트 구문의 종류를 구분해보면 다음과 같다.

  * &#8220;INSERT-like&#8221; statements 
      * 새로운 row를 생성하는 모든 구문 
          * `INSERT`
          * `INSERT ... SELECT`
          * `REPLACE`
          * `REPLACE ... SELECT`
          * `LOAD DATA`
      * 아래 명시된 모든 insert 종류들을 포함한다.
  * &#8220;Simple inserts&#8221; 
      * 몇줄이 insert될 지 실행전에 미리 **알 수 있는** 구문
      * 여러 줄을 한번에 insert 하더라도 미리 몇줄이 insert될지 알 수 있다면 bulk insert가 아닌 simple insert임에 주의
      * nested subquery가 없는 `INSERT`, `REPLACE` 
      * `INSERT ... ON DUPLICATE KEY UPDATE` 는 포함되지 않는다.
  * &#8220;Bulk inserts&#8221; 
      * 몇줄이 insert될지 미리 **알 수 없는** 구문 
          * `INSERT ... SELECT`
          * `REPLACE ... SELECT`
          * `LOAD DATA`
      * InnoDB가 한줄이 프로세싱될 때마다 AUTO_INCREMENT 컬럼의 값을 증가시켜줌
  * &#8220;Mixed-mode inserts&#8221; 
      * &#8220;simple insert&#8221;이긴 하지만 insert구문 내에 AUTO_INCREMENT 컬럼의 값을 **일부만 명시적으로 지정**해준 경우. (전체를 명시적으로 지정했다면 &#8220;simple insert&#8221;라고 볼 수 있음)
      * 예: `INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');`
      * `INSERT ... ON DUPLICATE KEY UPDATE` 
          * 상황에따라 AUTO_INCREMENT 컬럼에 할당된 값이 사용될 수도, 되지 않을 수도 있는 최악의 케이스

### 문제 상황

    Tx1: INSERT INTO t1 (c2) SELECT 1000 rows from another table ...
    Tx2: INSERT INTO t1 (c2) VALUES ('xxx');
    

위처럼 두개의 트랜잭션이 같은 테이블에 동시에 insert를 할때 AUTO_INCREMENT 컬럼의 값은 어떻게 증가되어야 하는가?

  * 테이블 레벨의 lock이 걸리는경우 Tx1이 끝날때까지 Tx2는 실행이 되지 않기때문에 동일 결과가 보장됨 
      * concurrency가 없기때문에 성능이 좋지 않다.
  * 테이블 레벨 lock이 아닌경우 Tx1의 insert가 진행중일때 Tx2가 같이 insert한다면? 
      * 실행되는 상황에따라서 AUTO_INCREMENT 값이 매번 달라질 가능성이 존재한다.
      * 이러한 상황이 문제가 되지 않는 경우를 잘 정의하면 좀더 concurrency를 높여서 성능을 좋게 할 수 있다.

위의 trade-off 관계를 정확히 이해하여 최고의 성능을 이끌어내기 위해 InnoDB에서 지원하는 `innodb_autoinc_lock_mode` 가 어떤 설정 값들을 지원하는지 아래에서 더 자세히 살펴보도록 하자.

## InnoDB가 제공하는 Lock Modes

  * traditional lock mode 
      * `innodb_autoinc_lock_mode = 0`
      * 모든 &#8220;INSERT-like&#8221; 구문에 대해서 table-level AUTO-INC lock을 적용 
      * 트랜잭션이 끝날때까지 적용되는 lock이 아닌 해당 구문의 실행시까지만 유지되는 lock
  * consecutive lock mode (기본 설정값) 
      * `innodb_autoinc_lock_mode = 1` 
      * &#8220;simple insert&#8221; 구문에 대해서는 table-level AUTO-INC locks을 회피할 수 있다.
      * AUTO_INCREMENT values의 할당 과정에서만 mutex를 이용해서 동시 접근을 제어하기 때문에concurrency를 높일 수 있다 (구문이 실행이 끝날때까지 table-level lock을 적용하는 것 보다 훨씬 효율적임)
      * insert시 생성되는 AUTO_INCREMENT 값들은 아래의 한가지 예외 경우를 제외하고는 traditional lock mode를 사용했을때와 결과값이 항상 동일하다.
      * 예외: &#8220;mixed-mode insert&#8221; 인 경우에 InnoDB는 실제 인서트 되어야 하는 rows 수 보다 더 큰 AUTO_INCREMENT 값을 할당한다. 지정된 값을 제외하고, 자동으로 할당된 값의 순차적인 증가는 보장된다.
  * interleaved lock mode 
      * `innodb_autoinc_lock_mode = 2` 
      * &#8220;INSERT-like&#8221; 구문에 대해서 table-level AUTO-INC lock을 사용하지 않는다.
      * concurrency가 가장 높지만 복구/복제용으로 SQL 바이너리 로그 replay를 사용하지 않고있는 경우에만 사용 가능
      * AUTO\_INCREMENT 값의 의 유니크성, 단조 증가성(monotonic increase)은 보장되지만 동일한 쿼리를 실행하더라도 실행 순서에 따라 매번 rows들이 가지는 AUTO\_INCREMENT 값들은 달라질 수 있다. (할당된 AUTO_INCREMENT 값이 구문들 간에 interleaved 될 수 있다.)
      * AUTO_INCREMENT 값의 Gap 존재 
          * &#8220;simple inserts&#8221;가 수행될때는 할당된 AUTO_INCREMENT 값들에 gap이 존재하지 않는다.
          * &#8220;bulk inserts&#8221;가 수행될때는 gap이 존재 할 수있다. 

## InnoDB의 AUTO_INCREMENT Counter 초기화 방식

InnoDB 테이블에 AUTO\_INCREMENT 필드를 추가하면, InnoDB의 테이블 정보 저장 공간에 AUTO\_INCREMENT counter를 보관한다.

  * 특이하게도 이 counter는 디스크가 아닌 메모리 상에만 보관된다. 
  * MySQL서버가 기동되고난 후 첫번째 insert구문 실행되는 시점에 InnoDB는 아래와 같은 구문을 먼저 실행해서 구한 값에서 1만큼 증가시킨 값을 counter를 메모리상에 로드한다. (auto\_increment\_increment이 따로 설정된 경우 해당 값 만큼 증가) 
        SELECT MAX(ai_col) FROM table_name FOR UPDATE;
        

아래 구문을 실행 할 때도 모든 테이블의 AUTO_INCREMENT 값들을 조회해 오게 되는데, 이 경우에는 각 테이블별로 현재 최대 값을 로드만 하고, 증가시키진 않는다.

    SHOW TABLE STATUS
    

이 상태에서 수동으로 AUTO_INCREMENT counter의 값을 수정하는 것도 가능하다. 수동으로 수정된 값이더라도, 서버가 재시작하면 다시 초기화 됨에 유의하자.

### AUTO_INCREMENT 카운터의 초기화가 끝난 후 INSERT시 동작

  * AUTO_INCREMENT 컬럼에 값이 명시되지 않은 insert를 실행한 경우: counter를 1 증가
  * AUTO_INCREMENT 컬럼에 값이 명시된 insert를 실행한 경우: 해당 값이 현재 counter 값보다 크다면 counter를 해당 값으로 업데이트

## 트랜잭션 상황에서 AUTO_INCREMENT 카운터 증가 실험

위에서 InnoDB가 제공하는 각각의 AUTO\_INCREMENT lock 모드에 대해 concurrent한 다양한 insert들이 일어날때에 대해서 어떻게 AUTO\_INCREMENT 값이 변화하는지는 자세히 정리해 보았다. 혹시나 transaction이 걸린 상태에서는 AUTO_INCREMENT 값의 변화가 달라지는지 확인을 해보기 위해서 아래와 같이 추가 실험을 해보았다.

### 실험 테이블 정의

CREATE TABLE `test` (  
`id` int(11) unsigned NOT NULL AUTO_INCREMENT,  
PRIMARY KEY (`id`)  
) ENGINE=InnoDB

### 상황 가정

  * 실행 전 AUTO_INCREMENT = 1 
  * Transaction isolation level = REPEATABLE READ
  * Transaction A, B는 동시에 진행 (각각 교차해서 statement를 실행)

| Transaction A               | Transaction B               | 쿼리 실행 후 AUTO_INCREMENT 값 |
| --------------------------- | --------------------------- | ------------------------ |
| start transaction;          | &#8211;                     | &#8211;                  |
| &#8211;                     | start transaction;          | &#8211;                  |
| INSERT INTO test VALUES (); | &#8211;                     | 2                        |
| INSERT INTO test VALUES (); | &#8211;                     | 3                        |
| &#8211;                     | INSERT INTO test VALUES (); | 4                        |
| &#8211;                     | INSERT INTO test VALUES (); | 5                        |
| INSERT INTO test VALUES (); | &#8211;                     | 6                        |
| commit or rollback          | commit or rollback          | &#8211;                  |

### 결과 분석

  * traditional, consecutive, interleaved 각 모드에 대해서 모두 동일한 결과를 얻음(AUTO_INCREMENT를 위해서 statement 단위 lock이 적용되기 때문인 듯)
  * 트랜잭션 A, B가 동시에 진행중일때 트랜잭션 내의 쿼리 실행 순서에 따라 AUTO_INCREMENT count가 각각 증가.
  * 트랜잭션 마지막에 rollback을 하던 commit을 하던 동일하게 증가된 AUTO\_INCREMENT count는 그대로 남아있음 (5 row를 insert 하고난 후에 rollback을 하더라도 AUTO\_INCREMENT count는 5가 증가한 상태 그대로)

INSERT 구문이 트랜잭션으로 묶여있는경우, 먼저 시작한 transaction의 INSERT가 완료 될 때 까지 늦게 시작한 트랜잭션의 AUTO\_INCREMENT 값이 제대로 반영되지 않을거라는 걱정이 있었지만, AUTO\_INCREMENT는 트랜잭션과는 완전 별개로 INSERT구문이 실행될 때마다 각각 증가함을 확인 할 수 있었다.

## 참고

https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html