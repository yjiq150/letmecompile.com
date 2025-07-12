---
title: 앱스토어 계정 이전시 애플 로그인 마이그레이션
author: yjiq150
type: post
date: 2023-09-13T01:40:23+00:00
url: /apple-sign-in-migration-on-appstore-app-transfer/
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/manage-aws-resources-with-pulumi-iac/"     class="post-1058"><span class="crp_title">Pulumi를 이용하여 코드로 AWS 리소스 관리하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/manage-aws-resources-with-pulumi-iac/"     class="post-1058"><span class="crp_title">Pulumi를 이용하여 코드로 AWS 리소스 관리하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part4/"     class="post-1024"><span class="crp_title">자바스크립트로 크롤러 만들기 4편: 실제 웹페이지 크롤링해보기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - iOS
  - 프로그래밍 기타
tags:
  - 앱스토어 계정이전
  - 앱 트랜스퍼
  - 앱 이전
  - 애플로그인 계정이전

---
<div class="wp-block-jetpack-markdown">
  <p>
    애플 로그인을 사용중인 앱을 다른 앱스토어 계정으로 이전하는 경우 해당 앱 내에서 애플 로그인 사용자를 구분하는데 사용되는 ID값(Team scoped identifier)이 변경된다. 이 값 뿐만 아니라 기존 애플 로그인 사용자가 이메일 가리기(private email) 기능을 활성화 했다면, 이로 인해 생성된 이메일 주소 또한 바뀌기 때문에 앱 이전(app transfer, 앱 트랜스퍼) 전/후로 별도의 마이그레이션을 꼭 수행해야한다. 이를 제대로 잘 처리하지않으면 기존 가입되어있던 사용자가 로그인을 시도해도 기존 사용자 정보로 로그인되는것이 아니고 신규 사용자 처럼 간주될 수 있기 때문에 주의해서 처리가 필요하다.
  </p>
  
  <p>
    앱 이전에 대한 전반적인 과정은 <a href="https://developer.apple.com/help/app-store-connect/transfer-an-app/overview-of-app-transfer/">이 공식 문서</a> 에 잘나와있으니 참고하시면 되고, 이 글에서는 전체 과정 중 &#8216;애플 로그인&#8217; 이 적용된 앱을 다른 계정으로 이전할 때 필요한 작업만을 집중적으로 다뤄보겠다.
  </p>
  
  <h2>
    앱 트랜스퍼 전 준비 작업
  </h2>
  
  <h3>
    Transfer identifier 저장
  </h3>
  
  <p>
    앞서 언급했던 것 처럼 앱 트랜스퍼 전/후로 사용자가 애플로그인을 했을 때, 사용자가 동일한 애플 ID로 로그인을 해도 트랜스퍼 전에 로그인 했을 때 받는 team scope identifier와 트랜스퍼 후에 로그인했을 때 받는 team scope identifier가 달라진다. 여기서 team scope identifier란 애플 로그인시 주어지는 token의 sub 필드 값으로 앱 내에서 애플 로그인 사용자를 구분하는데 사용되는 ID값을 의미한다. 이렇게 값이 달라지기 때문에 트랜스퍼 이전 계정의 team scope identifier와 트랜스퍼 이후의 team scope identifier를 이어줄 수 있는 맵핑 정보가 필요한데 이것이 바로 transfer identifier이다.
  </p>
  
  <p>
    transfer identifier를 획득하기 위해서는 아래 정보들이 필요하다.
  </p>
  
  <ol>
    <li>
      트랜스퍼 전 계정의 Team ID
    </li>
    <li>
      트랜스퍼 전 계정의 Apple Sign-in과 연결된 <code>.p8</code> Key 파일
    </li>
    <li>
      트랜스퍼되는 앱 번들ID
    </li>
    <li>
      트랜스퍼 후 계정의 Team ID
    </li>
  </ol>
  
  <p>
    transfer identifier를 획득하기위해 결국 애플의 migration API <code>/auth/usermigrationinfo</code> 를 호출해야하는데, 이 API를 호출할 권한을 얻기위해 인증 API <code>/auth/token</code>를 호출하여 액세스 토큰을 발급받아야한다.
  </p>
  
  <p>
    액세스 토큰을 발급받으려면 위 목록에서 1, 2, 3번 정보를 이용하여 <code>client_secret</code>을 생성한 후, 이를 이용하여 인증 API 를 호출하면 액세스 토큰을 응답으로 얻는다.
  </p>
  
  <p>
    이렇게 얻은 액세스 토큰을 넣어서 migration API를 호출하면 최종적으로 transfer identifier를 획득할 수 있고, 이 값을 유저DB에 저장해뒀다가 트랜스퍼 후에 사용하면 된다.
  </p>
  
  <p>
    위 과정은 말로 설명하면 어렵지만 실제 코드로 보면 오히려 더 간단하다.
  </p>
  
  <pre><code>async function main() {
    // TODO: Modify the following parameters to your own
    const sendingTeamId = 'S12341234P'
    const sendingTeamKeyId = 'T12341234L'
    const sendingTeamKeyFilePath = path.join(__dirname, 'Example_AuthKey_T12341234L.p8')
    const clientId = 'com.example.app'
    const receivingTeamId = 'R12341234P'

    // Just one appleId is used here for example. You usually should loop through all the users with Apple Sign-in.
    // Team scoped identifier for each user
    const appleId = '000506.5951a85d72c445918250badf39181d0f.0331'

    const clientSecret = generateClientSecret(sendingTeamId, sendingTeamKeyId, sendingTeamKeyFilePath, clientId)

    const accessToken = await getAppleApiAccessToken(clientId, clientSecret)

    const appleTransferId = await getAppleTransferId(clientId, clientSecret, accessToken, appleId, receivingTeamId)

    // Store this mappings in your user database before transfering app.
    console.log(`TransferID for ${appleId} is ${appleTransferId}`)
}
</code></pre>

<p>
  전체 코드는 <a href="https://github.com/yjiq150/apple-signin-migration-on-app-transfer">apple-signin-migration-on-app-transfer</a> 의 <code>before-app-transfer.js</code> 파일을 참고하면 된다.
</p>

<h3>
  로그인 코드 수정
</h3>

<p>
  앱 트랜스퍼가 완료되는 순간 직후부터 사용자가 애플 로그인을 하면 얻을 수 있는 토큰 내의 <code>sub</code> 필드에는 트랜스퍼된 계정 기준의 team scope identifier가 들어있고, 60일간 추가로 제공되는 <code>transfer_sub</code> 필드에는 좀 전 단계에서 저장해뒀던 transfer identifier가 들어있다.
</p>

<p>
  앱 트랜스퍼가 완료된 이후에는 뒤쪽에 설명할 &#8216;앱 트랜스퍼 후 마이그레이션&#8217; 작업을 통해 team scope identifier를 트랜스퍼된 계정 기준으로 모두 업데이트를 진행할 것이지만, 앱 트랜스퍼가 완료되는 순간에 이 업데이트를 모든 사용자에게 한번에 반영하는 것은 시간이 걸리기 때문에 매우 어렵다. (해당 업데이트가 진행중일때 기존 사용자가 애플 로그인 시도를 할 수 있기 때문이다. )
</p>

<p>
  따라서 로그인을 하는 순간 <code>transfer_sub</code> 값과 매칭되는 사용자를 DB에서 찾은 후 해당 사용자의 트랜스퍼 이전 team scope identifier를 트랜스퍼 이후 값으로 업데이트를 해주도록 미리 서버 코드를 수정해 줘야한다. 사용자가 이메일 가리기 기능을 활성화 한 경우 email 주소 또한 변경되기 때문에 이것도 같이 업데이트를 해주는 것을 잊지 말자.
</p>

<p>
  여기까지 작업했으면 모든 준비가 완료된 것이다. 이제 실제 앱스토어 커넥트를 통해 앱 트랜스퍼를 진행하면 된다.
</p>

<h2>
  앱 트랜스퍼 후 마이그레이션 작업
</h2>

<p>
  위에서 언급했던 애플 로그인시 제공되는 토큰 내의 <code>transfer_sub</code> 필드는 앱 트랜스퍼 후 60일간만 제공되고 그 이후에는 토큰에 더이상 포함되어 제공되지 않는다. 따라서 앱 트랜스퍼가 완료된 후 60일 동안 애플 로그인을 한번도 하지 않은 사용자는 제대로 마이그레이션이 되지 않는다. 이런 사용자들을 위해 서버에서 배치(batch) 작업을 통해 직접 애플의 migration API를 호출하여 트랜스퍼된 계정 기준의 team scope identifier를 받아서 사용자 정보를 일괄 업데이트 해주는 로직이 필요하다.
</p>

<p>
  트랜스퍼된 계정 기준으로 새롭게 사용자 정보를 획득하기 위해서는 아래 정보들이 필요하다.
</p>

<ol>
  <li>
    트랜스퍼 후 계정의 Team ID
  </li>
  <li>
    트랜스퍼 후 계정의 Apple Sign-in과 연결된 <code>.p8</code> Key 파일
  </li>
  <li>
    트랜스퍼된 앱 번들ID
  </li>
  <li>
    앱 트랜스퍼 전에 저장해둔 사용자별 transfer identifier
  </li>
</ol>

<p>
  새롭게 사용자 정보를 획득하기위한 과정 또한 기존 transfer identifier를 얻기위한 과정과 거의 동일하다. (트랜스퍼 후 계정의 정보들을 사용하는 것과 미리 저장해둔 transfer identifier를 활용하는 부분만 다르다.) 역시 애플의 migration API <code>/auth/usermigrationinfo</code> 를 호출해야하는데, 이 API를 호출할 권한을 얻기위해 인증 API <code>/auth/token</code>를 호출하여 액세스 토큰을 발급받아야한다.
</p>

<p>
  액세스 토큰을 발급받으려면 위 목록에서 1, 2, 3번 정보를 이용하여 <code>client_secret</code>을 생성한 후, 이를 이용하여 인증 API 를 호출하면 액세스 토큰을 응답으로 얻는다.
</p>

<p>
  이렇게 얻은 액세스 토큰을 넣고 특정 transfer identifier를 명시하여 migration API를 호출하면 트랜스퍼 후 계정 기준의 사용자 정보를 획득할 수 있고, 이 값을 이용하여 유저DB를 업데이트하면된다.
</p>

<p>
  위 과정 또한 실제 코드로 보면 오히려 더 간단하다.
</p>

<pre><code>async function main() {
    // TODO: Modify the following parameters to your own
    const receivingTeamId = 'R12341234P'
    const receivingTeamKeyId = 'S12341234L'
    const receivingTeamKeyFilePath = path.join(__dirname, 'Example_AuthKey_S12341234L.p8')
    const clientId = 'com.example.app'

    // Just one appleTransferId is used here for example. You usually should loop through all the users with Apple Sign-in.
    // Use 'transfer identifier' generated in 'before-app-transfer.js' for each user
    const appleTransferId = '000506.5951a85d72c445918250badf39181d0f.0331'

    const clientSecret = generateClientSecret(receivingTeamId, receivingTeamKeyId, receivingTeamKeyFilePath, clientId)

    const accessToken = await getAppleApiAccessToken(clientId, clientSecret)

    const { sub: appleId, email, is_private_email } = await getUserInfoByTransferId(clientId, clientSecret, accessToken, appleTransferId)

    // Update your user info in database with the new information.
    // When user enables private email for Apple Sign-in, the email is also different after transfer.
    console.log(`The new user information for ${appleTransferId} is ${appleId} / ${email} / ${is_private_email}`)
}
</code></pre>

<p>
  전체 코드는 <a href="https://github.com/yjiq150/apple-signin-migration-on-app-transfer">apple-signin-migration-on-app-transfer</a> 의 <code>after-app-transfer.js</code> 파일을 참고하면 된다.
</p>
</div>