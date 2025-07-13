---
title: 우분투 JVM Segmetation Fault 버그 해결 및 커널 업데이트 방법
date: 2018-04-03T15:33:53+00:00
url: /ubuntu-jvm-segmetation-fault-kernel-update/
dsq_thread_id:
  - 6592991654
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps

---
우분투(Ubuntu)의 커널 4.4.0.81 에서 JVM (Java Virtual Machine) 기반의 어플리케이션이 segmentation fault가 발생하면서 종료되어 버리는 현상이 존재한다.

JVM 기반의 프로그램이 이상하게 실행이 안되고 죽는다 싶으면 아래 명령어로 커널 버전을 확인하자

    uname -a 
    

출력된 커널 버전이 4.4.0.81이 맞다면 버그로 인해서 segmentation falut가 나는것이 확실하다.

  * 해당 버그 관련 참고 자료 
      * <https://usn.ubuntu.com/3344-1/>
      * <https://forum.ubuntuusers.de/topic/eclipse-crash-mit-speicherzugriffsfehler-nach-/>

### 해결 방법 1 (JVM 옵션 변경)

JVM 실행시에 `-Xss1280k` 옵션을 줘서 max stack size를 변경한다. JVM 에만 영향이있고 커널 업데이트를 하지 않아도 되서 부담이 없다. 하지만 계속 찝찝함이 남으니, 커널 업데이트가 가능한 상황이라면 다음의 커널업데이트를 통한 문제 해결 방법을 추천한다.

### 해결 방법 2 (커널업데이트)

우분투의 커널이 문제이니, 커널을 업데이트 하는 방법을 알아보도록 하자.

먼저 http://kernel.ubuntu.com/~kernel-ppa/mainline/ 사이트를 열어서 최신 버전 또는 원하는 버전의 커널을 선택하면 다음과 같은 화면이 뜬다.

![Screen Shot 2018-03-23 at 10.03.22 PM.png][1] 

여기서 32bits CPU의 경우 i386이 붙은 파일을, 64bits CPU의 경우 amd64가 붙은 파일을 다운로드 해야한다. (CPU가 인텔이냐 AMD인지와는 무관하다). wget, curl 등의 명령어를 사용하여 다운로드 하면 된다.  
그리고 &#8216;lowlatency&#8217;가 붙은 것만 다운로드 받을 필요가 없다.

예를들어 64bit CPU를 사용중이라면 아래 다섯개의 파일 중 볼드체로된 3개만 다운로드 하면 된다.

  * **linux-headers-4.15.11-041511\_4.15.11-041511.201803190530\_all.deb**
  * **linux-headers-4.15.11-041511-generic\_4.15.11-041511.201803190530\_amd64.deb**
  * linux-headers-4.15.11-041511-lowlatency\_4.15.11-041511.201803190530\_amd64.deb
  * **linux-image-4.15.11-041511-generic\_4.15.11-041511.201803190530\_amd64.deb**
  * linux-image-4.15.11-041511-lowlatency\_4.15.11-041511.201803190530\_amd64.deb

다운로드가 완료되면 위 세개 파일을 한 폴더에 모아두고 해당 폴더로 이동하여 다음 명령어를 실행한다.

    sudo dpkg -i linux*.deb
    rm linux*.deb
    sudo reboot
    

설치가 완료되면 재부팅이 시작되고 특별한 일이 없다면 무사히 커널업데이트가 완료되어 OS를 다시 정상적으로 사용할 수 있게 된다.

 [1]: https://steemitimages.com/DQmSTUXLDNrKzf9XxRMu9xryDBgdmVAND2At2csYQhtwy7r/Screen%20Shot%202018-03-23%20at%2010.03.22%20PM.png