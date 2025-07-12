---
title: 깃(Git) 에서 유실된 커밋(commit) 복원하기
author: yjiq150
type: post
date: 2018-03-11T15:03:34+00:00
url: /git-restore-lost-commits/
dsq_thread_id:
  - 6546334091
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 개발도구
tags:
  - git
  - 소스관리
  - 저장소
  - git복구
  - commit복구
  - 복구

---
깃(Git)을 이용하여 작업을 하다가 리베이스(rebase) 실수 또는 잘못된 명령어나 조작 실수 등 다양한 이유로 인해 자신의 피땀눈물이 담긴 커밋(commit)들을 날려먹는 경우가 은근히 있다. remote에 push해서 백업을 만들어놓고 로컬에서만 작업했으면 다시 remote 저장소에서 받아오면 되지만, 그렇지도 않은 경우에는 어떻게 이 지워진 커밋들을 다시 읽어오거나 복원할 수 있을까?

다행히도 한번이라도 commit이 된 내용이라면, 심지어 현재 보이는 git tree 상에 보이지 않는 commit들 까지도 local git repository안에 commit log들이 남아 있다. 덕분에 이를 검색해서 해당 commit 상태로 복원 할 방법이 존재한다.

git의 명령어 중에 reference logs 라는 의미를 가진 `reflogs` 라는 옵션을 커맨드라인에서 실행해보자.

    git reflog
    

위 명령어가 실행되면 나오는 화면에서 저장소 tree에 일반적으로 보이지 않는 모든 commit들을 살펴볼 수 있다. 여기서 유실된 commit을 찾은 후 해당 commit의 commitID를 찾아서 상황에 맞게 다음 명령어들을 사용하면 된다.

해당 유실된 커밋을 HEAD로 하는 tree로 돌려놓으려면 아래처럼 리셋 명령어를 실행하면된다.

    git reset --hard {commitID}
    

해당 유실된 커밋만 현재 브랜치로 가져오려면 아래처럼 체리픽을 이용한다.

    git cherry-pick {commitID}