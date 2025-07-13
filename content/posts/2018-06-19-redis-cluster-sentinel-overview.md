---
title: 레디스 클러스터, 센티넬 구성 및 동작 방식
date: 2018-06-19T00:29:54+00:00
url: /redis-cluster-sentinel-overview/
dsq_thread_id:
  - 6740328417
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - DevOps
  - Database

---
RDBMS만큼의 정합성과 영속성을 보장할 필요가 없는 데이터들을 빠르게 처리하거나 일정 기간동안만 보관하고 있기 위한 용도로 레디스(Redis), memcached 등의 in-memory 기반 저장소가 많이 사용된다. 그중에서도 Redis는 빠른 성능을 유지하면서도 일정 수준 이상의 persistence를 제공하고, 다양한 native 데이터 구조들을 제공하여 효율적이고 편리한 사용이 가능하게 해주기 때문에 다양한 use-case들이 존재한다.

이 글은 실제 명령어를 날려서 레디스를 직접 사용해보는 것을 배우기 보다는 어떤 in-memory 저장소를 선택할지 고민하는 분들을 위해서 주요 운영 방식, 레디스의 내부 동작 방식 및 특징, 주요 클라이언트들에 대한 정보를 제공하는 쪽에 초점이 맞춰져 있다.

## 운영 모드 (Operation Modes)

레디스는 단일 인스턴스만으로도 충분히 운영이 가능하지만, 물리 머신이 가진 메모리의 한계를 초과하는 데이터를 저장하고 싶거나, failover에 대한 처리를 통해 HA를 보장하려면 센티넬이나 클러스터 등의 운영 모드를 선택해서 사용해야 한다. 각 모드들이 어떤 특성을 갖는지 좀더 자세히 알아보도록 하자.

  * 단일 인스턴스(Single instance) 
      * HA(High availibilty) 지원 안됨.
  * 센티넬(Sentinel) 
      * HA 지원
      * master/slave replication
      * sentinel process 
          * redis와 별도의 process
          * 여러개의 독립적인 sentinel process들이 서로 협동하여 운영된다 (SPOF 아님)
          * 안정적 운영을 위해서는 최소 3개 이상의 sentinel instance 필요 (fail over를 위해 과반수 이상 vote 필요) 
              * redis process가 실행되는 각 서버마다 각각 sentinel process를 띄워놓는 방법
              * redis process가 실행되는 서버와 별개로 redis에 액세스를 하는 application server들에 sentinel process를 띄워놓는것도 가능
              * 등등 다양한 구성이 가능
          * 지속적으로 master/slave 가 제대로 동작을 하고있는지 모니터링
          * master에 문제가 감지되면 자동으로 failover 수행
      * 클라이언트는 어떻게 redis server에 연결해서 데이터를 조회하나? 
          * 먼저 sentinel에 연결해서 현재 master를 조회해야 한다.
  * 클러스터(Cluster) 
      * HA, sharding 지원 
          * Sentinel과 동시에 사용하는 것이 아님! 완전히 별도의 솔루션.
          * dataset을 자동으로 여러 노드들에 나눠서 저장해준다.
          * Redis Cluster 기능을 지원하는 client를 써야만 데이터 액세스 시에 올바른 노드로 redirect가 가능하다.
      * Cluster node들의 동작 방식 
          * serve clients 6379 (data port)
          * cluster bus 16379 (data port + 10000) 
              * 자체적인 바이너리 프로토콜을 통해 node-to-node 통신을 한다.
              * failure detection, configuration update, failover authorization 등을 수행
          * 각 노드들은 클러스터에 속한 다른 노드들에 대한 정보를 모두 갖고있다.
      * Sharding 방식 
          * 최대 1000개의 노드로 샤딩해서 사용. 그 이상은 추천하지 않음
          * consistent hashing을 사용하지 않는대신 hashslot이라는 개념을 도입
          * hashslot 
              * 결정방법 CRC16(key) mod 16384를 
                  * CRC16을 이용하면 16384개의 슬롯에 균일하게 잘 분배됨
              * 노드별로 자유롭게 hash slot을 할당 가능
              * 예) 
                  * Node A contains hash slots from 0 to 5500.
                  * Node B contains hash slots from 5501 to 11000.
                  * Node C contains hash slots from 11001 to 16383.
          * 운영 중단 없이 hash slots을 다른 노드로 이동시키는 것이 가능 
              * add/remove nodes
              * 노드별 hashslot 할당량 조정
          * multiple key operations 을 수행하려면 모든 키값이 같은 hashslot에 들어와야 한다. 
              * 이를 보장하기위해 hashtag 라는 개념 도입 
                  * {} 안에있는 값으로만 hash 계산
                  * `{foo}_my_key`
                  * `{foo}_your_key`
      * Replication & failover 
          * failover를 위해 클러스터의 각 노드를 N대로 구성가능 
          * master(1대) / slave(N-1대)
          * async replication (master → slave replication 과정에서 ack을 받지 않음) 
              * 데이터 손실 가능성 존재
              * master가 client요청을 받아서 ack을 완료한 후, 해당 요청에 대한 replication이 slave로 전파되기 전에 master가 죽는 경우 존재
      * 클라이언트는 클러스터에 어떻게 연결해서 데이터를 조회하나? 
          * redis client는 클러스터 내의 어떤 노드에 쿼리를 날려도 된다(슬레이브에도 가능). 
              * ex) GET my_key
          * 쿼리를 받은 노드가 해당 쿼리를 분석 
              * 해당 키를 자기 자신이 갖고있다면 바로 찾아서 값을 리턴
              * 그렇지 않은경우 해당 키를 저장하고 있는 노드의 정보를 리턴 (클라이언트는 이 정보를 토대로 쿼리를 다시 보내야함)
              * ex) `MOVED 3999 127.0.0.1:6381`

## 메모리 동작 방식

  * key가 만료되거나 삭제되어 redis가 메모리를 해제하더라도, OS에서 해당 분량만큼 바로 메모리가 확보되진 않음 
      * 꼭 redis에만 해당되는 이야기는 아님
      * 5GB중 3GB의 데이터를 메모리에서 해제 -> OS 메모리 사용량은 여전히 5GB
      * 하지만 다시 데이터를 add하면 logically freed된 영역에 잡히므로 실제 메모리 5GB를 넘지는 않는다.
  * _따라서 peak memory usage 기준으로 잡아야 한다._
  * 대부분 5GB 정도 사용하고 가끔 10GB 가 필요하더라도, 10GB 메모리를 이상의 머신이 필요.
  * maxmemory 설정을 해두는게 좋음 (하지 않으면 무한히 메모리 사용하다가 머신 전체가 죽을 가능성) 
      * maxmemory 설정시의 eviction policy 
          * no-eviction (추가 memory를 사용하는 write command에 대해 에러 리턴)
          * allkeys-lru (전체 아이템 중에서 LRU)
          * volatile-lru (expire 되는 아이템 중에서 LRU)
          * volatile-ttl (expire 되는 아이템 중 TTL이 얼마 안남은 녀석 순으로)
      * RDB persistence를 사용한다면 maxmemory를 실제 가용 메모리의 45%정도로 설정하는것을 추천. 스냅샷을 찍을때 현재 사용중인 메모리의 양만큼 필요하다. (5%는 오버헤드에 대비한 마진)
      * 사용하고 있지 않다면 가용 메모리의 95%정도로
  * 동일 key-value 데이터를 저장한다고 가정했을 때, Cluster Mode를 사용할 경우 Single instance 보다 1.5~2배 정도 메모리를 더 사용하는것에 주의해야 한다. 
      * Redis cluster의 경우 내부적으로 cluster안에 저장된 key를 hashslot으로 맵핑하기 위한 테이블을 가지고 있기 때문에 추가적인 memory overhead가 발생한다.
      * 이때문에 key의 숫자가 많아질수록 이러한 현상이 더 두드러진다
      * 4.x 버전에 이와 관련한 메모리 최적화 기능이 들어가서 3.x 버전보다는 더 적게 메모리를 사용하지만, 여전히 Single instance보다는 많은 메모리를 필요로 한다.

## 데이터 영속성 (Data Persistence)

memcached의 경우 데이터가 메모리에만 저장되기 때문에 프로세스가 재기동되면 메모리상의 데이터는 모두 유실된다. 하지만 redis의 경우 기본적으로 disk persistence가 설정되어있기 때문에, 프로세스를 재시작 하더라도 셧다운 되기 전의 마지막 상태와 거의 동일한 (약간의 손실은 있을 수 있다) 상태로 돌려 놓을 수 있다.

  * RDB persistence 
      * 일정 인터벌로 point-in-time snapshots을 생성
      * compact한 단일 파일로 저장됨 백업하기 좋음
      * 부모 프로세스는 자식 프로세스를 fork. fork된 프로세스에서 모든 persist I/O처리
      * restart 시간이 빠르다 
          * H/W 사양에 따라 다르겠지만 보통 메모리 사용량 1GB당 10~20 초정도 로드타임
      * 멈췄을때 데이터 유실 가능성이 더 높다.
      * 몇 분 정도의 데이터 유실 가능성을 감내 할 수 있다면 RDB를 쓰는것을 추천
  * AOF (append only file) 
      * 모든 write operation 을 log로 남기고 서버 재시작 시점에 replay
      * 데이터 양이 많아지면 해당 시점의 데이터셋을 만들어낼 수 있도록하는 minimum log들만 남기는 compaction을 진행
      * 읽기 쉬운 포멧이지만 RDB보다 용량이 크다
      * write 리퀘스트가 많을때 RDB보다 반응성이 느리다
  * 참고: http://oldblog.antirez.com/post/redis-persistence-demystified.html

라인 메신저의 메시징 시스템에서는 RDB또는 AOF 사용으로 인해

## 트랜잭션 모델(Transaction model)

  * Redis는 single threaded model이고 요청이 들어온 순서대로 처리한다
  * `MULTI` → commands → `EXEC`/`DISCARD`
  * `MULTI` 이후의 명령들은 queue에 넣어뒀다가 `EXEC`가 불린순간 순차적으로 진행 
      * `EXEC`를 통해서 실행중일때는 다른 connection에서 중간에 끼어들지 못한다
  * command 실행중에 에러가 발생해도 롤백하지 않고 계속 진행한다. 
      * command에 잘못된 명령어나, 잘못된 타입의 인자를 넣었을때 에러 발생 -> 거의 개발자의 실수
      * 롤백기능을 없앤 덕분에 훨씬 빠른 성능 제공 가능
  * optimistic locking (using CAS operation) 사용 
      * 값의 변경을 모니터링하다가, 값이 변경되었다면 현재 트랜잭션을 취소하고 다시 처음부터 실행
      * `WATCH` 
          * 특정 키값이 변경되지 않은 경우에만 `EXEC`를 수행, 변경된경우 transaction자체를 수행하지 않음
          * `EXEC`가 불리는 시점에 모든 key에 대한 `WATCH`가 자동으로 `UNWATCH`됨
          * client의 연결이 끝나는 시점에도 모든 key에 대해 `UNWATCH`됨
      * 예시) 
          * WATCH mykey
          * val = GET mykey
          * val = val + 1
          * MULTI
          * SET mykey $val
          * EXEC

## 주요 특수 기능

  * 다양한 데이터 구조 지원 
      * 단순히 key &#8211; value 문자열만 저장하는 것이 아니라 고수준의 데이터 구조를 사용 가능하다
      * ex) Hash, Set, List, SortedSet, etc.
      * Hash 
          * HSET(key, fields, value), HGET(key, field)
          * web application에서 특정 유저 userId를 key로 두고 해당 유저의 세부 정보들(name, email 등)을 field로 둔다
          * 이렇게하면 특정 유저와 관련된 정보들을 한번에 삭제하는 등의 namespace처럼 사용하는것도 가능하다.
          * hash key당 1000개 정도의 field까지는 레디스가 zipmap을 이용해 압축해서 저장한다 
              * 동일한 value 1000개를 저장할때 방식에 따른 메모리 사용량을 비교 
                  * “1000개의 키에 저장” > “1개의 키 안에 100개의 field로 저장”
              * 대신 CPU사용량이 증가하므로 어플리케이션 특성에 맞게 적정 갯수를 잘 선택해야한다.
              * 참고: https://instagram-engineering.com/storing-hundreds-of-millions-of-simple-key-value-pairs-in-redis-1091ae80f74c
              * 참고: https://redis.io/topics/memory-optimization
  * Expiration 지정 
      * key별로 TTL(Time-To-Live)을 정해두면 레디스가 알아서 해당 시점이 지날때 key 삭제
      * 설정된 max memory에 도달하면 expire되지 않은 key들도 eviction policy에 따라 삭제될 수 있다.
  * Pipelining 
      * 여러 커맨드들을 한번에 모아서 보낸후, 실행 결과도 한번에 모아서 받는다.
      * 레이턴시가 큰 네트워크 환경이라면 명령어 하나당 한번의 request/response를 할 때보다 스루풋을 올릴 수 있음.
  * Pub/Sub (출판/구독 모델) 
      * 하나의 클라이언트가 같은 채널에 연결된 다른 클라이언트들에게 메시지들을 보내는 것이 가능
      * 이를 이용하여 속도가 빠른 메시지 브로드캐스터 혹은 메시지 큐 처럼 사용하는것이 가능하다.
  * Lua scripting 
      * 여러 명령어들이 사용되는 복잡한 작업을 할 때 (특히 트랜잭션이 사용되는 경우) 이를 하나의 lua script로 만들어서 사용할 수있다.
      * 스크립트는 atomic하게 실행되기 때문에 optimistic locking transactions 를 사용할 필요가 없다.

## 지표 모니터링 (Monitoring Metrics)

  * used_memory
  * total\_commands\_processed 
      * 이 지표와 요청/응답 latency 로깅해두면 명령어 처리량 대비 latency를 알 수 있다.
      * redis-cli -h {host} -p {port} —latency
  * slow command 
      * slowlog get 명령으로 확인
  * client connections 
      * info clients 명령으로 확인
      * 대략 5000개 이상의 커넥션들이 존재한다면 응답 시간이 느려지고 있을 가능성이 있으니 주의깊게 모니터링
      * maxclients 값을 대략 예상되는 connection 숫자의 1.1배~1.5배정도로 설정해 두면 안전함
  * mem\_fragmentation\_ratio 
      * 정의 = 실제 물리 메모리 사용량 / 레디스의 메모리 사용량
      * 1 이면 이상적인 상태, 커질수록 파편화 심화
      * 1.5가 넘는다면 서버 재시작 추천
      * 1보다 작은 경우 
          * 레디스의 메모리 사용량이 실제 물리 메모리 사용량보다 큰 경우는 OS에 의해 메모리의 일부가 swap되어 디스크로 이동되었을때 나타남
          * 디스크 사용으로 인해 성능 저하 발생
          * 메모리 사용량을 줄이거나 램 용량 증설 필요
  * 참고: https://www.datadoghq.com/pdf/Understanding-the-Top-5-Redis-Performance-Metrics.pdf

## 클라이언트 핸들링(Client Handling)

  * maxclients 설정 가능 
      * 설정된 maxclient 도달시에 
  * timeout default = forever
  * Redis 3.2 이상부터 TCP keep-alive 디폴트로 켜져있음(약 300초)

## Redis Client library for Java

언어별로 레디스 클라이언트 라이브러리들이 다양하게 존재하고 있다. 그중에서 자바언어로 만들어진 가장 많이 사용되는 3가지를 뽑아서 비교해 보았다.  
async style API를 사용하는 경우 처리량(throughput)을 높일 수 있지만 오히려 개별 요청에 대한 응답 지연(latency)은 sync API 보다 더 느려질 수 있으니 상황에 맞게 선택해서 사용하는 것이 좋다.

  * jedis 
      * https://github.com/xetorthio/jedis 
      * light & simple (sync API only)
      * low level 명령을 직접 실행
      * connection pool
      * master에만 명령 실행 가능 (slave 선택 불가)
      * http://www.baeldung.com/jedis-java-redis-client-library
  * redisson 
      * https://github.com/redisson/redisson
      * async & sync API (netty기반이라 좀 헤비하다)
      * high-level abstraction 명령어가 아닌 객체를 조작 
          * 예) Map이나 Set 객체를 생성해서 사용하면 해당 객체가 Redis서버에 자동으로 싱크가됨 (데이터 마샬링 필요 없음)
          * jedis 방식: redis get → java model object → redis put
          * redisson 방식: get RObject → modify RObject (modification sync to redis internally by redisson)
      * https://stackoverflow.com/questions/42250951/redisson-vs-jedis-for-redis
  * lettuce 
      * https://lettuce.io/
      * async & sync API (netty기반이라 좀 헤비하다)
      * read only 명령어에 대해서는 master/slave에 선택적으로 실행 가능

## 참고자료

  * <https://redis.io/documentation>
  * [개발 언어별 유명 redis client 비교 분석][1]
  * [Redis vs. memcached 비교 분석 (stackoverflow)][2]
  * <https://www.datadoghq.com/pdf/Understanding-the-Top-5-Redis-Performance-Metrics.pdf>
  * <https://engineering.linecorp.com/ko/blog/detail/306>

 [1]: https://www.compose.com/articles/a-tour-of-the-redis-stars-2/
 [2]: https://stackoverflow.com/questions/10558465/memcached-vs-redis/11257333