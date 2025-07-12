---
title: 카프카(Kafka) Consumer offset reset 방법
author: yjiq150
type: post
date: 2018-08-31T06:54:43+00:00
url: /kafka-consumer-offset-reset/
dsq_thread_id:
  - 6885027808
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps
  - Server

---
카프카 Consumer를 사용하다 보면 offset을 reset해야하는 경우가 종종 있다.

  * 개발 테스트를 진행하다가 필요에의해 offset을 리셋
  * 실제 production에서 사용중에 예상치 못한 에러 등으로 데이터 누락이 발생하여 일정기간 전으로 다시 offset을 rewind해서 사용하고자 하는 경우 

이런 경우에 Consumer API를 사용중이라면 직접 코드 레벨에서 programmatic하게 reset도 가능하고, 아니면 kafka에서 제공하는 reset tool을 이용해서 reset이 가능하다.

## 카프카 바이너리 설치

카프카 바이너리를 설치한다. Mac에서 brew가 있다면 아래 명령어로 간단히 설치 할 수 있다.  
(Mac을 기준으로 설명 한다.)

    brew install kafka
    

Kafka 0.10.x 버전 이하의 경우에는 아래 사용할 명령어들이 존재하지 않는다. 따라서 설치된 kafka 버전이 0.11.x 이상임을 확인하자. (brew의 경우 현재 기본적으로 2.0 이상이 설치된다.)

## Consumer group의 offset 상태 확인

consumer group을 지정하고 `--describe` 옵션을 사용하면 현재 consumer group의 offset 정보를 볼 수 있다. 명령어는 다음과 같다.

    kafka-consumer-groups --bootstrap-server <host:port> --group <group.id> --describe
    

실행결과는 다음과 같다.

    TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                      HOST            CLIENT-ID
    example.topic 0          6392623366      6392623859      493             consumer-1-f6f6ffb0-1054-46b9-af13-0b254bc14da0  /10.64.69.95    consumer-1
    example.topic 1          6394637143      6394637383      240             consumer-10-6c57b320-7742-4418-8e15-b7d735da346e /10.64.69.95    consumer-2
    example.topic 2          6397170269      6397170495      226             consumer-19-dbed41a1-42bb-4ecb-bc8f-84e47c74dbe8 /10.64.69.95    consumer-3
    example.topic 3          6397170269      6397170495      226             consumer-19-dbed41a1-42bb-4ecb-bc8f-84e47c74dbe8 /10.64.69.95    consumer-4
    

결과로 출력된 각 컬럼을 간단하게 설명하면,

  * TOPIC: 토픽 이름
  * PARTITION: consumer group 내의 각 consumer가 할당된 파티션 번호
  * CURRENT-OFFSET: 현재 consumer group의 consumer가 각 파티션에서 마지막으로 offset을 commit한 값
  * LOG-END-OFFSET: producer쪽에서 마지막으로 생성한 레코드의 offset
  * LAG: LOG-END-OFFSET에서 CURRENT-OFFSET를 뺀 값. 

`--describe`를 통해 조회를 했을때 LAG이 계속 일정 수준을 유지한다면 consumer가 producer 가 만들어내는 이벤트 레코드의 양을 잘 따라가고있다는 것을 확인할 수 있다. 하지만 LAG이 계속 증가한다면 consumer의 처리 속도가 느린 것이기 때문에 consumer의 갯수를 충분히 증가시키거나, consumer의 로직을 더 간략화 해서 빠른 속도로 데이터 처리를 할 수 있도록 변경해야 한다.

## Consumer group의 offset reset

kafka에서 데이터를 불러와서 처리하는 과정에서 오류가 발생하거나 문제가 발견된 경우, 다시 원하는 offset부터 데이터를 재처리를 해야할 경우가 종종 있다. 이때 consumer group의 offset reset 기능을 활용하면 된다.

**주의사항**: consumer group이 실행중인 상태에 offset reset을 진행하는 경우 reset은 실패한다.

    kafka-consumer-groups --bootstrap-server <host:port> --group <group> --topic <topic> --reset-offsets --to-earliest --execute
    

`--topic` 대신 `--all-topics`를 지정하면 모든 토픽에 대해서 실행이 가능하다.

`--execute` 옵션을 제거하고 실행하면 실제 반영되지 않고 어떻게 변할지 결과만 출력하는 dry run이 가능하다.

오프셋의 위치를 재설정하기 위한 아래와같은 상세 옵션들이 있다.

  * `--shift-by <Long: number-of-offsets>` 형식 (+/- 모두 가능)
  * `--to-offset <Long: offset>`
  * `--to-current`
  * `--by-duration <String: duration>` : 형식 &#8216;PnDTnHnMnS&#8217;
  * `--to-datetime <String: datetime>` : 형식 &#8216;YYYY-MM-DDTHH:mm:SS.sss&#8217;
  * `--to-latest`
  * `--to-earliest`

`--to-datetime`의 경우 kafka에서 데이터를 읽어서 다른곳에 저장하는 중에 데이터 유실 또는 중복 write 등이 발생한 경우에 날짜 단위로 데이터를 다시 불러와서 재처리하고 싶은 경우 매우 유용하다.

## CLI가 아닌 Java 코드로 offset reset하기

Kafka의 경우 사용 형태에 따라 Consumer API와 Stream API 두가지를 제공한다.

  * Consumer API 
      * 개별 이벤트 단위의 low level 처리가 필요한 경우
      * datetime, offset 을 지정해서 원하는 대로 reset 가능
  * Stream API 
      * Stream processing이 필요한 경우
      * offset reset 기능 없음 (위에서 언급한 CLI tool을 사용해야함)

Consumer API를 사용할때 Java코드 레벨에서 programmatical하게 offset을 리셋하는 방법은 다음과 같다.  
먼저 KafkaConsumer가 생성한 후에

    KafkaConsumer<Object, Object> consumer = new KafkaConsumer<>(properties, keyDeser, valueDeser);
    

consumer loop에 진입하여 consumer.poll()을 부르기 전에, 생성된 consumer 객체에 대해 offset을 변경하는 다음 함수들을 호출하여 offset을 원하는 대로 설정할 수 있다.

  * `seekToBeginning`: earliest로 reset
  * `seekToEnd`: latest로 reset
  * `seek` : 지정 offset으로 reset
  * `offsetsForTimes`: datetime 기준으로 reset

### Reference

<https://gist.github.com/marwei/cd40657c481f94ebe273ecc16601674b>

<http://blog.sysco.no/integration/kafka-rewind-consumers-offset/>