---
title: 작업 스케쥴러 크론(Cron) 간단 사용법
author: yjiq150
type: post
date: 2018-03-23T13:17:51+00:00
url: /scheduler-cron-tutorial/
dsq_thread_id:
  - 6571878129
dsq_needs_sync:
  - 1
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/verify-domain-setting-changes/"     class="post-701"><span class="crp_title">도메인 설정 변경 확인 명령어</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 미분류

---
unix 계열 운영체제 (linux, MacOS 포함)를 사용하다보면 시간대별로 반복되는 작업들을 수행해야 하는 경우가 매우 자주 있다. 이때 작업들의 스케쥴링을 위해서 application을 데몬처럼 실행시켜두고 해당 application 안에 통해서 스케쥴링을 할수도 있지만, 이미 unix 계열 운영체제에 기본으로 설치되어있는 cron 을 이용하면 훨씬 더 간편하게 특정 작업들을 실행할 수 있다.

## 크론 스케쥴 편집하기

아래 명령어를 실행하면 현재 로그인된 유저에 대해 cron job을 등록 할 수 있다.

    crontab -e 
    

만약 특정 작업이 root 권한이 필요하여 root 유저에 대해 cron job을 등록하고 싶다면 아래 명령어를 이용하면 된다.

    sudo crontab -e
    

명령어를 실행하면 디폴트 text 편집기가 열리면서 cron의 설정파일을 수정할 수 있게된다. 이 편집화면에서 원하는대로 설정을 변경한 후 저장하면 된다.

![cron_sample.png][1] 

## 작업 스케쥴 설정

    * * * * * 명령어
    

작업 스케쥴을 설정할때는 순서대로 min > hour > day(일) > month > weekday(요일)를 한칸씩 띄워서 적어주면 된다.

예를들어 `40 5 * * * /some/job` 의 경우 job을 매 05시 11분에 실행하라는 뜻이다.

좀더 다양한 예시들을 살펴보자.

  * `* * * * *` 1분 마다 실행
  * `30 * * * *` 매시 30분마다 실행
  * `0 0 10 * *` 매월 10일 0시 0분에 실행
  * `* * 10 * *` 매월 10일에 1분 마다 실행
  * `0 0 10 4 *` 매년 4월 10일에 0시 0분에 실행
  * `0 0 * * 1` 매주 월요일 0시 0분에 실행 (요일의 숫자표현: 일0 월1 화2 수3 목4 금5 토6)
  * `0 * * * 1` 매주 월요일 매시 0분에 실행

시간 포멧은 <https://crontab.guru/>과 같은 사이트를 이용하면 좀더 쉽게 보면서 만들어낼 수 있다.

## 작업에 대한 로그(Log)를 남기는 방법

cron job들은 자동으로 수행되기때문에 별도로 작업 로그를 남기지 않으면 해당 작업이 정상적으로 수행되었는지 체크하기가 매우 까다롭다. cron job의 수행 여부 자체는 시스템 로그에 (Ubuntu의 경우 `/var/log/syslog` 파일을 확인하면 됨) 남기때문에 볼 수 있지만, 실제 해당 job이 실행되면서 job에서 발생시키는 로그는 따로 기록을 해주어야한다.

cron job을 등록할 때 아래와 같은 명령어를 사용하면 로그 파일을 남길 수 있다.

    0 0 * * * /some/job > ~/log/job_`date +\%Y\%m\%d\%H\%M\%S`.log 2>&1
    

위 명령어가 잘 이해가 가지 않는 사람들을 위해 좀더 자세하게 설명을 해보려고 한다.

### 로그파일의 파일이름에 실행된 날짜/시간을 넣는 방법

`date` 명령어를 이용하면 지정한 포메터 대로 현재 시간을 출력할 수 있다. 쉘 스크립트에서 &#96; 문자를 이용하여 &#96;명령어&#96; 를 감싸면 해당 명령어의 stdout을 return해주는데 이를 이용하여 파일이름을 정해줄 수 있다.

로그 파일명을 다음과 같이 지정한 경우 job\_&#96;date +\%Y\%m\%d\%H\%M\%S&#96;.log 실제 파일명은 job\_20180309201123.log 와 같이 시간값으로 치환되어 결정된다.

### `/some/job`의 출력(stdout)을 로그 파일로 리디렉션

리디렉션 기호인 `>`를 이용하여 stdout을 파일로 기록하려면 다음과 같이 하면 된다.

해당 로그 파일에 overwrite를 하고싶다면

    /some/job > ~/log/job_`date +\%Y\%m\%d\%H\%M\%S`.log  
    

해당 로그파일의 기존 내용을 보존하면서 append를 하고싶다면

    /some/job >> ~/log/job_`date +\%Y\%m\%d\%H\%M\%S`.log  
    

### `/dev/null`의 의미?

위에서 알아본 것과 반대로 작업 로그가 필요 없다고 생각되는 경우는 /dev/null 로 redirect를 해주면 된다. `/dev/null` 로 들어오는 입력은 모두 버려진다.

### 마지막 `2>&1`의 의미?

  * `1`: stdout
  * `2`: stderr
  * `>`: redirection
  * `2>&1`: stderr를 stdout으로 리디렉션해서 stdout과 동일하게 처리

 [1]: /uploads/2018/03/cron_sample.png