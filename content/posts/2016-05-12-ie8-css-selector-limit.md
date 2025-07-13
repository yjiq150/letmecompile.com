---
title: IE8 CSS selector limit
date: 2016-05-12T00:23:22+00:00
url: /ie8-css-selector-limit/
dsq_thread_id:
  - 4819368559
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-transaction-model/"     class="post-766"><span class="crp_title">MySQL InnoDB Transaction Model 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발

---
IE8의 경우 css 파일하나당 셀렉터 4095개 갯수제한이 들어가있다.

css 파일 내에 셀렉터의 총 갯수가 4095개가 넘는 순간 해당 css파일은 IE8에의해 로딩되지 않고, 렌더링에 사용되지 못하기때문에 페이지 레이아웃이 전부 깨지게 된다.

일반적으로 파일하나에 4095개가 넘는 css를 정의하는 경우는 많지 않지만, `grunt`나 `gulp`등의 빌드 엔진의 플러그인들을 통해 여러개의 css파일들을 하나로 병합(merge)해서 사용하는 경우 문제가 될 소지가 다분하다.  
많이 쓰이는 `bootstrap`의 css 셀렉터 갯수만 해도 약 3000개가 넘어가기때문에, `bootstrap`과 다른 몇몇 css만 병합해도 바로 문제가 발생하니 주의해서 사용해야 한다.

따라서 IE8을 지원하려면 merge된 css의 셀렉터 갯수를 체크해서 문제가 없는지 확인해야한다.  
[selector counter][1]

사용해 보진 않았지만 자동으로 css셀렉터 갯수를 세어서 자동으로 쪼개주는 도구도 존재하니 필요한경우 사용하면 좋을것 같다. <http://blesscss.com>

 [1]: http://snippet.bevey.com/css/selectorCount.php