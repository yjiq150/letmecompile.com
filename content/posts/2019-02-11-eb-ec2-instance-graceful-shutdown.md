---
title: Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정
author: yjiq150
type: post
date: 2019-02-10T16:16:34+00:00
url: /eb-ec2-instance-graceful-shutdown/
dsq_thread_id:
  - 7222922147
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/aws-application-load-balancer-custom-error-message/"     class="post-832"><span class="crp_title">AWS Application Load Balancer Custom Error Message</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/aws-application-load-balancer-custom-error-message/"     class="post-832"><span class="crp_title">AWS Application Load Balancer Custom Error Message</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps
  - AWS

---
AWS에서 Elastic Beanstalk나 EC2에서 직접 오토스케일링 설정해서 사용하다보면 트래픽이 늘었을 때 인스턴스의 갯수가 늘어났다가(scale out) 트래픽이 줄어들면 불필요해진 자동으로 인스턴스를 shutdown하게 된다(scale in). 이때 shutdown 될 인스턴스에 연결되어서 진행중인 요청(in-flight request)이 있다면 어떻게 될까?

먼저 해당 인스턴스로 새롭게 추가 요청이 가지 않도록 연결된 로드밸런서(Load Balancer, LB)에서 shutdown 될 등록 해제 프로세스(Deregistration process)를 시작한다. 이때 LB에서 인스턴스로의 연결이 열려있는것이 있다면 설정된 시간동안 연결이 자연스럽게 종료 되도록 기다려준다. (이를 connection draining 또는 deregistration delay라고 부른다.) 이 시간을 초과하면 열려있는 연결이 있더라도 강제로 종료한 후 LB에서 완전히 제외되며 인스턴스를 종료시킨다.

만약 이런 절차를 거치지 않고 바로 종료해 버린다면, 정상적으로 요청 중이던 유저는 에러를 겪게 될 것이기 때문에 일반적으로 서버가 처리하는 요청들 중 소요되는 시간이 가장 긴 요청을 기준으로 잡아서 넉넉히 시간을 설정해 주는 것이 좋다.

## 설정 방법

Classic LB와 Application LB의 모두 같은 기능을 제공하는데 용어가 다르고 설정하는 곳도 다르니 주의가 필요하다.

### AWS EC2 Console을 사용하는 경우

  * EC2 > Load Balancing 메뉴 
      * Classic LB 인 경우 
          * Load Balancers > LB 선택 > Instances 탭에서 Connection Draining 수정
      * Application LB인 경우 
          * Target Groups > Target Group 선택 > Deregistration delay를 수정

### Elastic Beanstalk를 사용하는 경우

deploy때마다 설정이 일관되게 유지되게 하려면 deploy되는 application 파일 안에 설정을 포함시키는게 좋다.

application의 .ebextensions 폴더에 config파일을 만들고 다음 내용을 추가한다.

  * Classic LB인 경우 
        option_settings:
          - namespace: aws:elb:policies
            option_name: ConnectionDrainingEnabled
            value: true
          - namespace: aws:elb:policies
            option_name: ConnectionDrainingTimeout
            value: 300
        

  * Application LB인 경우 
        option_settings:
          - namespace: aws:elasticbeanstalk:environment:process:default
            option_name: DeregistrationDelay
            value: 300  
        

### 참고

  * <https://aws.amazon.com/ko/blogs/aws/elb-connection-draining-remove-instances-from-service-with-care/>
  * <https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/load-balancer-target-groups.html>
  * <https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/load-balancer-target-groups.html>
  * <https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/command-options-general.html>