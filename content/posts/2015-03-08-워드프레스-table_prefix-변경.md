---
title: 워드프레스 table_prefix 변경
author: yjiq150
type: post
date: 2015-03-07T15:31:40+00:00
url: /워드프레스-table_prefix-변경/
dsq_thread_id:
  - 3575602310
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-utf8-utf8mb4-migration/"     class="post-691"><span class="crp_title">MySQL utf8에서 utf8mb4로 마이그레이션 하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 워드프레스 개발

---
기존에 설치하여 사용하고 있던 워드프레스에서 wp-config.php 파일의 $table_prefix를 변경하려면 다음 스텝에따라 진행하자.

본 포스팅에서는 다음 상황을 가정하고 설명할 예정이다.

  * 기존 $table\_prefix = &#8216;wp\_&#8217;
  * 변경된 $table\_prefix = &#8216;newprefix\_&#8217;

## 1. DB 테이블명 변경

DB에 접속해서 변경된 prefix를 가지도록 DB명을 변경한다. DB명을 변경 후 워드프레스 사이트에 로그인 해보면 정상적인 로그인이 되고 사이트가 작동하는것을 볼 수 있다.  
하지만 기존 사용자들의 capability와 role이 인식이 제대로 되지 않는것을 발견 할 수 있다.

이를 고치기 위해서 2번 스텝을 진행한다.

## 2. 기존 table_prefix와 관련된 테이블 내 값 변경

옵션테이블과 유저메타 테이블에 프리픽스와 연관된 항목이 있기때문에 이 항목의 키값들을 새 프리픽스를 가지도록 변경해주어야 한다.

  * newprefix_options 테이블 수정 
      * `option_name` 컬럼이 `wp_user_roles` 이라고 되어있는 옵션값이 존재한다. 이 값을 `newprefix_user_roles` 로 변경해야한다.
      * 다음 쿼리를 실행하면 된다. 
             UPDATE newprefix_options SET option_name='newprefix_user_roles' WHERE option_name='wp_user_roles';
            

  * newprefix_usermeta 수정 
      * `meta_key` 컬럼이 `wp_user_level` 이라고 되어있는 메타값이 존재한다. 이 값을 `newprefix_user_level` 로 변경해야한다.
      * `meta_key` 컬럼이 `wp_capabilities` 이라고 되어있는 메타값이 존재한다. 이 값을 `newprefix_capabilities` 로 변경해야한다.</p> 
      * 다음 쿼리를 실행하면 된다.
        
             UPDATE newprefix_usermeta SET meta_key='newprefix_user_level' WHERE meta_key='wp_user_level';
             UPDATE newprefix_usermeta SET meta_key='newprefix_capabilities' WHERE meta_key='wp_capabilities';
            

이제 워드프레스 사이트에서 확인해보면 정상적으로 권한이 잘 동작하고 있는것을 확인 할 수 있다.

WordPress capability error after table prefix renamed