---
title: AWS EC2 ssh 로그인 tips
author: yjiq150
type: post
date: 2015-03-15T01:58:03+00:00
url: /aws-ec2-ssh-login-tips/
dsq_thread_id:
  - 3596042294
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 맥 Tips
  - 웹 개발
  - DevOps
tags:
  - ssh 연결을 이용해 GUI 에디터로 파일 편집하기
  - How do you edit files over SSH
  - use gui editor with ssh
  - AWS 관리

---
AWS EC2 인스턴스 관리에 있어서 public/private key를 이용한 ssh 인증을 잘 이용하면 높은 보안성과 동시에 비밀번호 입력이 필요없는 편리한 환경을 구축 할 수 있다.

## AWS에 ssh 로그인 편리하게 하기

AWS(Amazon Web Service) EC2 계정 생성과정에서 필수적으로 public/private key pair를 생성하게 되는데 이를 통해서 인스턴스 접근 인증을 하게된다. 생성시점에 public key는 AWS에 자동으로 저장되고 EC2 console 화면의 Key Pairs 메뉴에서 등록된 public key를 확인할 수 있다. private key의 경우는 생성시점에 `.pem` 파일 형태로 다운로드를 해서 저장하게되는데 이 키파일에 대한 보관 책임은 사용자에게 있다. 이 key pair 를 통해 로그인 인증을 하게되므로 항상 이 키파일을 보관하는데 주의를 기울여야 한다.

### 1. 기본 로그인 방법

AWS instance를 생성한 후 처음으로 인스턴스의 shell에 접속하기 위해서는 다음과 같은 명령어가 필요하다.  
`private_keyfile.pem` 은 AWS EC2 처음 계정생성시 만들어진 private 키 파일이다.

    ssh -i <private_keyfile.pem> <yourAccountID>@<Public IP Address>
    

### 2. 키 파일 지정 없이 로그인

키파일 지정없이 ssh로 로그인을 시도할경우 다음과 같은 에러를 만나게 된다.

> Permission denied (public key) 

해당 에러는 현재 컴퓨터의 기본 public 키가 인스턴스상에 저장이 되어있지 않기때문에 인증을 할 수 없어서 나타나는 오류이다. 이를 해결하기 위해 다음과같은 절차를 진행한다.

자신이 사용하는 로컬 컴퓨터의 `~/.ssh/id_rsa.pub` 파일의 내용을 복사  
(기존해 생성해둔 id_rsa 파일이 존재하지 않을경우 `ssh-keygen` 명령어를 통해 생성하면 된다.)

기본 방법을 이용해 AWS 인스턴스로 로그인 후, `~/.ssh/authorized_keys` 에 붙여넣고 저장

AWS 인스턴스의 `authorized_keys` 파일에 자신이 사용하는 로컬컴퓨터의 퍼블릭 키가 저장된 이후로는 ssh 커맨드에 키파일을 직접 지정하지 않아도 해당컴퓨터에서는 비밀번호 없이 바로 로그인이 가능해진다.

    ssh <yourAccountID>@<Public IP Address>
    

## Mac에서 AWS 의 파일 시스템 마운트해서 쓰기

[osxfuse/sshfs 다운로드][1]

위 주소에서 osxfuse와 `sshfs`를 설치하면 리모트(remote) 환경에있는 리눅스 머신의 파일 시스템을 마운트해서 편리하게 사용가능하다.

`sshfs` 명령어는 커맨드라인 인터페이스라 사용이 좀 까다로울 수 있으니 [MacFusion][2]을 사용하여 GUI환경에서도 편리하게 mount/unmount 하는것을 추천한다. MacFusion의 경우에는 ssh연결시 키파일을 따로 지정하는 옵션이 없어서 맥에 저장된 기본 키파일을 사용하게되므로 `2. 키파일 지정없이 로그인` 셋업을 해두어야 연결이 가능하다.

마운트가 된 이후에는 파인더에서 바로 AWS에 있는 파일을 읽고/쓰는 것이 가능해진다. vi같은 편집기에 익숙하지 않을경우 맥에서 실행가능한 GUI 텍스트 편집 툴을 사용해서 바로 작업이 가능하므로 작업효율이 매우 올라 간다.

## 외부에서 AWS 인스턴스의 MySql 연결하기

일반적으로 MySql 은 외부 접속을 막아두지만 ssh를 통해 먼저 로그인 후 해당 shell에서 mysql을 접근하는것이 가능하다. 맥용 GUI DB 관리툴인 [Sequal Pro][3] 에서 이 기능을 지원하는데, 연결 설정에서 SSH를 선택하여 사용하면 된다.

 [1]: http://osxfuse.github.io/
 [2]: http://macfusionapp.org/
 [3]: http://www.sequelpro.com/