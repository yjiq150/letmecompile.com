---
title: EC2 Snapshot 주기적으로 백업하기
date: 2019-02-11T01:28:02+00:00
url: /ec2-snapshot-주기적으로-백업하기/
dsq_thread_id:
  - 7223883493
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps
  - AWS

---
<https://github.com/colinbjohnson/aws-missing-tools/tree/master/ec2-automate-backup>

위 스크립트를 이용하여 다음과 같은 명령어를 cron이나 jenkins job으로 등록하여 매일 한번씩 호출되게 해준다.

    ec2-automate-backup.sh -n -p -r ${region} -v ${ebs_volume_id} -k ${delete_after_days}
    

  * -n 스냅샷 백업시에 태그에 있는 내용들을 name 필드에도 설정
  * -p 스냅샷에 태깅된 시간보다 현재시간이 더 큰경우 스냅샷 삭제(purge)
  * -r 지역 설정 (ex: ap-northeast-1)
  * -v 백업하려는 EBS volume ID (ex: vol-00cb123456789) EC2 콘솔에서 확인 가능
  * -k 스냅샷 생성시 며칠 후 까지 보관하면될지 시간을 태깅 (ex: 7, 7일)

예를 들어, 아래와같은 스크립트를 매일 한번 실행하면, ap-northeast-1에 있는 vol-00cb123456789 스토리지에 대한 스냅샷을 매일만들면서, 7일 이상 된 스냅샷은 자동으로 삭제해 준다.

    ec2-automate-backup.sh -n -p -r ap-northeast-1 -v vol-00cb123456789 -k 7
    

`ec2-automate-backup.sh` 스크립트는 내부적으로 `aws` CLI 명령어를 사용하기 때문에, AWS CLI 설정이 미리 되어있어야 제대로 동작한다. 이를 설정하는 방법에 대해서는 [AWS CLI 구성 가이드][1]에 잘 나와있으니 참고하면 된다.

AWS CLI 구성이 완료되었어도, 해당 AWS 유저에게 EC2관련 권한이 없는경우 권한 오류가 발생할 수 있으니 IAM에서도 EC2관련 권한들을 추가해 줘야한다.

 [1]: https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-configure.html