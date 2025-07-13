---
title: Jenkins에 저장된 credential/password 값 알아내기
date: 2021-10-27T14:12:30+00:00
url: /jenkins에-저장된-credential-값-알아내기/
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/keep-process-after-disconnected-from-ssh-shell/"     class="post-817"><span class="crp_title">SSH shell에서 실행중인 작업을 연결 종료 후에도 유지하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/keep-process-after-disconnected-from-ssh-shell/"     class="post-817"><span class="crp_title">SSH shell에서 실행중인 작업을 연결 종료 후에도 유지하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 미분류

---
젠킨스에 크레덴셜로 저장된 비밀번호 등의 값을 알아내고 싶은 경우 두가지 방법이 있다

1. job을 만들어서 아래 파이프라인 스크립트를 수행

<https://www.codurance.com/publications/2019/05/30/accessing-and-dumping-jenkins-credentials> 내용을 기반으로 좀더 간단하게 작성했음

<pre class="wp-block-code"><code>pipeline {
  agent any
  stages {
    
    stage('sshUserPrivateKey') {
      steps {
        script {
          withCredentials(&#91;
            sshUserPrivateKey(
              credentialsId: 'gerrit',
              keyFileVariable: 'keyFile',
              passphraseVariable: 'passphrase',
              usernameVariable: 'username'
              )
          ]) {
            print 'keyFile=' + keyFile
            print 'passphrase=' + passphrase
            print 'username=' + username
            print 'keyFile.collect { it }=' + keyFile.collect { it }
            print 'passphrase.collect { it }=' + passphrase.collect { it }
            print 'username.collect { it }=' + username.collect { it }
            print 'keyFileContent=' + readFile(keyFile)
          }
        }
      }
    }
  }
}</code></pre>

2. jenkins 서버에 직접 ssh 접속이 가능하다면 /var/jenkins_home/credential.xml 파일을 오픈 (젠킨스 설치 경로는 설치 방식에 따라 조금씩 다를 수 있음)

파일 안에서 원하는 password 또는 credential을 찾음. (암호화되어 저장되어있는 경우 `{XXXXX==}` 와 같은 식으로 표현되어있음)

https://your.jenkins.com/script 경로에 접근한 후에 해당 창에 아래와 같이 원하는 암호화된 값을 넣고 실행하면 복호화된 값이 출력됨

<pre class="wp-block-code"><code>println(hudson.util.Secret.decrypt("{XXXXX==}"))</code></pre>