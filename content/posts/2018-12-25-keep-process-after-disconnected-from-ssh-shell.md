---
title: SSH shell에서 실행중인 작업을 연결 종료 후에도 유지하기
date: 2018-12-25T13:49:52+00:00
url: /keep-process-after-disconnected-from-ssh-shell/
dsq_thread_id:
  - 7128137180
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/ubuntu-jvm-segmetation-fault-kernel-update/"     class="post-732"><span class="crp_title">우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 프로그래밍 기타
  - DevOps

---
원격 서버에 ssh로 접속한 후 실행시간이 긴 명령어를 수행중에 ssh 연결을 끊거나 네트워크 오류로 인해 연결이 끊어지게되면 실행중이던 명령어도 같이 종료되어버린다. ssh를 통해 실행된 shell에서 실행한 프로세스는 shell의 child 프로세스로 연결되어있기 때문에 이런 현상이 발생하는데, 이를 방지하려면 child 프로세스를 parent에서 detach하여 독립된 프로세스로 만들면 된다. 이를 위해 해당 명령어를 `nohup` 등의 명령어를 이용할 수도 있지만 `tmux` 명령을 사용하면 가장 손쉽게 해결 할 수 있다.

## 설치

Ubuntu에서는 다음 명령어로 설치가 가능하고, 다른 계열의 OS에서는 해당 OS에맞는 패키지 매니저를 사용하면 된다.

    sudo apt-get install tmux
    

## 사용 방법

`tmux`를 실행하면 새로운 쉘 세션이 시작된다. 여기에서 원하는 명령어를 실행한 후 `CTRL` + `b` 를 누르고 바로 `d` 키를 누르면 해당 실행중인 명령어는 계속 실행 중인 상태를 유지하면서 (shell에서 detach 됨) 다시 `tmux` 쉘 세션으로 빠져나오게 된다. 이 상태로 `tmux` 쉘 또는 현재 ssh 연결 자체를 종료해도 실행했던 프로세스는 쉘로 부터 detach 되었기 때문에 해당 프로세스의 작업이 종료되어 끝날 때 까지 계속 실행중인 상태를 유지한다.

이때 다시 해당 프로세스로 연결하고 싶다면 `tmux attach` 명령을 수행하면 된다.

## 요약

  * tmux 실행
  * tmux 세션 안에서 명령어 실행
  * `ctrl` + `b` 누른 후 `d`
  * tmux attach