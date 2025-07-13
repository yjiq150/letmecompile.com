---
title: PHP 날짜/시간 Timezone, TimeStamp, DateTime 관계
date: 2014-10-02T14:32:09+00:00
url: /php-날짜시간-timezone-timestamp-datetime-관계/
dsq_thread_id:
  - 3077301908
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/kerberos-protocol/"     class="post-737"><span class="crp_title">커버로스 프로토콜(Kerberos Protocol) - 서버 접근 권한 관리</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발

---
PHP를 DateTime 클래스를 다루다 보면 입/출력에따라 타임존 오프셋(timezone offset)이 반영이 되는지 안되는지 헷갈리는 경우가 꽤 많은데 아래 두가지 principle만 알면 큰 실수는 하지 않을 수 있다.

  1. &#8216;now&#8217; 등의 기능을 이용하여 현재시간을 타임스탬프로 얻어온다면 항상 이값은 UTC+0 기준 현재시간에 대한 타임스탬프이다. (PHP 디폴트 타임존과 무관)
  2. PHP에 디폴트 타임존이 설정된 상태로 DATETIME 스트링을 이용하여 DateTime 객체를 초기화 하면 해당 객체의 타임스탬프 값은 타임존 오프셋이 이미 반영된 값이다.

아래 예시를 보면 더 명확히 이해가 갈것이다.

## 입력 DATETIME 스트링이 동일하고 타임존이 다른 경우 결과

    $date1 = new DateTime('2014-10-01 20:00:00', new DateTimeZone('Asia/Seoul'));
    $date2 = new DateTime('2014-10-01 20:00:00', new DateTimeZone('Europe/London'));
    // 날짜 스트링은 동일하지만, 시간대가 다르므로 타임스탬프에 타임존offset이 반영되서 나옴.
    echo $date1->getTimeStamp(); // 1412161200 // UTC 11시의 timestamp (타임스탬프는 항상 UTC 기준이기 때문)
    echo $date2->getTimeStamp(); // 1412193600 // UTC 20시의 timestamp
    
    // 출력시에는 입력한 스트링이 동일하게 나옴
    echo $date1->format("Y-m-d H:i"); // 2014-10-01 20:00:00 그대로 출력
    echo $date2->format("Y-m-d H:i"); // 2014-10-01 20:00:00 그대로 출력
    

## 입력 TimeStamp값이 동일하고 타임존이 다른 경우 결과

    $date1 = new DateTime('now', new DateTimeZone('Asia/Seoul'));
    $date2 = new DateTime('now', new DateTimeZone('Europe/London'));
    // 동일한 타임스탬프값 출력됨
    echo $date1->getTimeStamp(); // 1412193600 
    echo $date2->getTimeStamp(); // 1412193600
    
    // 타임스탬프 기준으로 타임존이 반영된 로컬타임이 출력됨
    echo $date1->format("Y-m-d H:i"); // 2014-10-02 05:00:00 출력 (UTC+9 기준 로컬타임이 출력됨)
    echo $date2->format("Y-m-d H:i"); // 2014-10-01 20:00:00 출력