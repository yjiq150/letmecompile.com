---
title: MySQL utf8에서 utf8mb4로 마이그레이션 하기
author: yjiq150
type: post
date: 2017-11-14T18:14:06+00:00
url: /mysql-utf8-utf8mb4-migration/
dsq_thread_id:
  - 6283866584
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 프로그래밍 기타
  - 웹 개발

---
Emoji같은 글자들은 utf8 인코딩 되는경우 글자당 최대 4bytes까지 필요하다. 하지만 기존 MySQL의 utf8 필드의 경우 글자당 최대 3bytes 까지만 지원하는 한계점이 있었다. 때문에 MySql database에서 utf8mb4 설정을 해두지 않으면 해당 글자들이 포함된 텍스트가 입력되었을때 제대로 저장을 하지못하고 문자가 깨져버리는 사태가 발생하게 된다. 최근들어 이러한 글자들의 사용이 보편화 되고있기 때문에 이러한 문제를 막기 위해서 서버 환경변수와 데이터베이스 테이블 스킴을 utf8mb4를 지원하도록 어서 변경을 해두는 것이 좋다.

😁😂😃😄😅😆💩

**주의: 항상 그렇듯이 모든 변경 전에는 DB를 백업해두자!**

## MySql Server 환경 변수 변경

AWS RDS를 사용하는 경우 RDS 콘솔에 접근한 후 파라메터 그룹(Parameter groups) 화면에서 서버 환경변수들을 설정해주면 된다. 환경변수를 설정 한 후 인스턴스 수정화면에 가서 파라메터 그룹을 바꿔주면되는데, 변경 후 인스턴스를 재시작 해야하는 점에 주의하자.

    character_set_client     = utf8mb4            
    character_set_connection = utf8mb4            
    character_set_database   = utf8mb4            
    character_set_filesystem = binary             
    character_set_results    = utf8mb4            
    character_set_server     = utf8mb4            
    character_set_system     = utf8               
    collation_connection     = utf8mb4_unicode_ci 
    collation_database       = utf8mb4_unicode_ci 
    collation_server         = utf8mb4_unicode_ci
    

  * 참고: 아래 두 설정값은 아래의 설정값이 맞으며 utf8mb4로 변경할 필요가 없다. 
      * character\_set\_filesystem = binary
      * character\_set\_system = utf8

MySql server를 직접 운영하는 경우 my.conf 파일을 적절히 수정하여 재시작 하면 된다. (이부분은 직접 해보진 않아서.. 자세한 내용은 이곳을 참고)<sup id="fnref-691-change"><a href="#fn-691-change" class="jetpack-footnote">1</a></sup>

### collation 설정에 대하여

collation은 &#8220;책 페이지의 순서 조사&#8221; 라는 사전적의미를 갖고 있는데 database에서는 문자열을 정렬할때 어떤 문자들이 먼저올지를 결정하는 기준을 말한다. 예를들어 알파벳 문자를 정렬할때 순서를 AaBb 순으로 해야할지 ABab순으로 해야할지 생각 하기에 따라 기준이 달라질 수 있는데, 이러한 미묘하게 다른 기준들이 각각의 collation에 정의되어있다. <sup id="fnref-691-query"><a href="#fn-691-query" class="jetpack-footnote">2</a></sup>

collation의 경우 utf8mb4\_general\_ci 이 기본값이다. 이 설정을 사용할경우 정렬 속도가 utf8mb4\_unicode\_ci 설정에 비해 약간 빠르다고는 하는데 (거의 미미하다고 함) 한글이나 일본어같은 비 라틴계 언어들에 대해서 조금 어색한 정렬 순서들이 존재한다고 한다. 때문에 utf8mb4\_unicode\_ci를 사용하는 것이 더 좋고, 이를 위해서 추가 설정을 해주어야한다.

### 변경된 설정 확인하기

아래 쿼리를 실행하여 환경 변수들이 잘 적용되었는지 살펴본다.

    SHOW GLOBAL VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
    

## 데이터베이스 테이블 변경

아래는 MySQL의 시스템 테이블중 하나인 information_schema에서 테이블 정보를 불러와서 table alter쿼리를 생성하는 SQL구문이다.

    USE information_schema;
    
    SELECT concat("ALTER DATABASE `",table_schema,
                  "` CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;") as _sql
      FROM `TABLES`
     WHERE table_schema like "데이터베이스 이름"
     GROUP BY table_schema;
    
    SELECT concat("ALTER TABLE `",table_schema,"`.`",table_name,
                  "` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;") as _sql
      FROM `TABLES`
     WHERE table_schema like "데이터베이스 이름"
     GROUP BY table_schema, table_name;
    

&#8220;데이터베이스 이름&#8221; 부분에 자신의 데이터베이스 이름을 넣어주고 실행을 하면 table alter query들이 잔뜩 생성된다. 이렇게 생성된 쿼리들을 다시 실행해주면 각 테이블에 존재하는 모든 문자필드의 인코딩 타입 변경된다. utf8는 utf8mb4의 서브셋이기때문에 인코딩 타입이 변경되었다고해서 해당 필드에 저장된 문자열이 깨지지 않는다.

다만 문자 하나당 최대 bytes수가 바뀌면서 문자열 필드에 인덱스가 걸려있었다면 문제가 될 가능성이 있다. InnoDB 엔진의 경우 인덱스 할 수있는 최대 크기는 767 bytes 이다. 따라서 아래의 계산법에 따라 인덱스가 걸린 VARCHAR(255)인 필드가 있었다면 이 필드는 utf8mb4로 변경이 불가능하고 VAR(191)로 길이를 줄여줘야 인덱스를 유지한 상태로 utf8mb4로 변경이 가능하다.

  * VARCHAR(255)이 차지하는 최대 bytes 수 
      * utf8인 경우 255 x 3 = 765 bytes (인덱스 가능)
      * utf8mb4인 경우 255 x 4 = 1020 bytes (인덱스 불가능!)
  * VARCHAR(191) 
      * utf8mb4인 경우 191 x 4 = 764 bytes (인덱스 가능)

<li id="fn-691-change">
  https://mathiasbynens.be/notes/mysql-utf8mb4&#160;<a href="#fnref-691-change">&#8617;</a>
</li>
<li id="fn-691-query">
  https://dba.stackexchange.com/questions/8239/how-to-easily-convert-utf8-tables-to-utf8mb4-in-mysql-5-5&#160;<a href="#fnref-691-query">&#8617;</a> </fn></footnotes>