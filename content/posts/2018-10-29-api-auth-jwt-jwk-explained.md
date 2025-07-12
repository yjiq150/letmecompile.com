---
title: API 서버 인증을 위한 JWT와 JWK 이해하기
author: yjiq150
type: post
date: 2018-10-28T16:35:21+00:00
url: /api-auth-jwt-jwk-explained/
dsq_thread_id:
  - 7002179744
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발
  - Server

---
쿠키(cookie)를 이용한 세션기반의 인증의 경우 특정 웹서버에서 세션 상태(session state)를 유지해야 하기 때문에 stateless 하지않다. 서버 로직이 Stateless가 아닌 경우 더 많은 요청을 처리하기 위해 동일한 서버의 숫자를 늘리는 스케일 아웃(scale out)에 적합하지 않다. 또한 도메인이 다른 서버에 대해서는 해당 세션 쿠키가 공유되지 않기 때문에 도메인이 다른 서버에 요청하기 위해서는 매번 새롭게 인증을 해야하는 불편함도 존재한다.

이 문제를 해결하기 위해 매번 http 요청마다 http header에 인증 토큰(authorization token)을 같이 보내는 형태의 방법을 많이 사용한다. 일반적으로 토큰 안에는 어떤 유저가 보내는 요청인지 구분하기 위해서 유저 ID값이 포함된다. 이 방법을 사용하면 서버쪽에서 세션 상태를 유지할 필요가 없어서 스케일 아웃에 적합하며, 도메인이 다른 서버에 요청하는 경우에도 동일한 토큰을 그대로 사용할 수 있다. 이러한 토큰기반의 인증을 적용하는 경우 악의적인 유저가 다른 유저ID를 사칭하는 것을 방지하기 위해서 토큰에 서명(signature)을 포함하거나, 대칭키 암호화를 적용한다.

이러한 방식은 API 서버를 개발하는 많은 사람들이 공통으로 많이 사용하다 보니 중복된 개발을 막기 위해서 JWT (JSON Web Token)라는 표준이 만들어지게 되었다. 실제로 직접 API 서버를 개발하거나 Auth0, AWS Cognito 등의 인증 서비스를 제공하는 플랫폼에서도 JWT가 많이 사용되고있다.

## JWT (JSON Web Token)

  * 표준화: [RFC 7519][1]
  * JWT는 header, payload, signature 각각 base64 encoding 한 후 concat한 문자열이다. 
      * 토큰에 포함된 내용들은 암호화되지 않아서 누구나 확인 할 수 있다.
      * 토큰에 포함되는 여러 종류의 필드들이 용도별로 미리 정의 되어있어서 일관성 있는 사용이 가능하다.
      * signature를 이용하여 해당 토큰이 실제로 원래 발급자가 발급했던 유효한 토큰인지 검증을 할 수 있다.
      * signature 생성을 위한 알고리즘을 원하는 대로 선택이 가능하다. (ex: RS256, HS256 등)
      * 실제 생성된 JWT 스트링의 샘플은 [https://jwt.io][2] 에서 손쉽게 확인 가능하다.

### JWT의 각 요소 설명

  * header 
      * signature 생성을 위해 사용한 알고리즘을 명시 (ex: RS256)
      * key rolling을 지원하는 경우, 존재하는 여러개의 key 중 어떤 key를 signature를 생성 할 때 사용했는지 알기 위해 kid (key 별로 unique한 값이 정의되어 있음)를 포함하기도 한다.
  * payload 
      * JSON key-value 형태로 데이터들이 포함되어있으며, 이곳에 포함된 각 필드를 클레임(Claim)이라고 부른다.
      * 예시) JWT를 유저인증을 위해 사용하는 경우 payload에 토큰을 발급받은 사용자의ID 값을 `aud` 필드에 포함하면 되고, 다음과 같은 절차를 따른다. 
          * JWT를 이용하여 application server에 요청
          * server에서 JWT의 signature 유효성을 확인하고
          * 유효 하다면 payload에서 사용자 ID값을 읽어들여서 요청을 보낸 사람이 어떤 사용자 인지 인증 할 수 있다.
  * signature 
      * 설명을 간단히 하기위해 header와 payload를 concat 한 값을 message라고 하자.
      * 해당 message가 변조되지 않았는지 검사하기 위해 서명이 필요하고, 아래 두가지 방식이 대표적이다. 더 보안성을 좋게하기 위해서 비트(bit) 수를 늘려 256 대신 384나 512를 사용하기도 한다.
      * RS256 (RSA Signature with SHA-256) 
          * 비대칭키 방식
          * message에 SHA256적용 후 private key 사용해서 암호화
          * JWT를 발급한 서버뿐만아니라, JWT를 받아서 사용하는 어떤 주체라도 signature 유효성 검증이 가능
          * 일반적으로 public key는 JWT를 발급한 서버에서 JWK (JSON Web Key)에 정의된 방식을 통해 공개적으로 제공
      * HS256 (HMAC with SHA-256) 
          * 대칭키 방식
          * message에 SHA256 적용 후 대칭키 사용해서 암호화
          * JWT를 발급한 서버 또는 해당 대칭키를 미리 공유해서 알고있는 주체들만 signature 유효성 검증이 가능하다. 

### 참고: key rotation (또는 key rolling) 이란?

  * 보호하려고 하는 데이터가 모두 하나의 key로 암호화 되어있는 경우 해당 key가 유출되면 모든 데이터가 유출되게 된다. 이러한 위험성을 줄이기 위해서 위해서 주기적으로 key를 변경하는 것을 key rotation 이라고 한다. 이 경우 특정 key가 유출되더라도, 특정 기간동안에 생성된 데이터만 유출되고, 다른 key로 암호화된 데이터는 안전하다.</p> 
  * 일반적인 인증서의 경우 1~2년 정도의 유효기간을 가지게 된다. 인증서 마이그레이션을 위해 보통 구버전의 인증서가 만료되는 시점과 새로운 인증서의 유효기간을 겹치도록 새 인증서를 발급하고, 겹치는 기간동안은 두 인증서를 모두 지원해야 한다. 결국 인증서도 public key, private key의 조합이기 때문에 이러한 절차도 key rotation이라고 볼 수 있다.

## JWK (JSON Web Key)

  * 표준화: [RFC 7517][3]
  * JWK는 암호화 키를 표현하기 위한 다양한 정보를 담은 JSON 객체에 관한 표준이다.
  * JWT를 사용하는 AWS Cognito, Auth0 서비스들을 보면 JWT를 서명하는데 사용했던 public key를 제공하기 위해 JWK 형태로 표현된 key를 접근할 수있는 URL을 제공한다. 서비스에 정의된 URL에 접근하면 JWK 형태로 key를 다운로드 할 수 있다. 
      * 예시) <https://cognito-idp.ap-northeast-2.amazonaws.com/ap-northeast-2_zkRIo1CpV/.well-known/jwks.json>

### JSON 객체에 포함되는 주요 필드 설명

  * kty (key type) 
      * 사용되는 값 
          * RSA
          * EC(Elliptic Curve)
      * REQUIRED
  * use (Public Key Use) 
      * 퍼블릭 키가 어떤 용도로 사용되는지 명시
      * 사용되는 값 
          * sig (signature)
          * enc (encryption)
      * OPTIONAL
  * alg (algorithm) 
      * 어떤 알고리즘을 적용하는데 이 key를 사용할 것인지를 명시 
          * ex) RS256
      * OPTIONAL
  * kid (key ID) 
      * key rolling이 일어나는 시점에는 여러개의 key들이 동시에 존재할 수 있다. 이때 각 key를 구분짓기 위해서 unique한 값으로 설정하여 사용한다. 
          * 이 경우 JWT token의 header영역에 해당 JWT를 생성할 때 사용한 kid 값을 넣어두고, signature verify하는 과정에서 JWK의 meta endpoint에서 제공하는 여러개의 key들 중 kid가 같은 것을 찾아서 사용한다.
      * key가 여러개가 아닌경우 필요 없는 경우도 존재하기 때문에 OPTIONAL
  * n: RSA modulus
  * e: RSA public exponent
  * x5c: x509 certificate chain
  * x5t: x.509 certificate thumbprint (SHA-1)

### 참고: RSA key의 구성 요소

  * RSA public key는 modulus와 public exponent로 구성되어 있다. 때문에 `n`, `e` 값이 주어지면, public key가 주어진 것이나 마찬가지이다.
  * private key에는 `n`, `e` 정보가 둘다 있기때문에 private key만 있으면 public key를 추출해 낼 수 있다.
  * RFC 3447에 정의된 [ASN.1 형식][4]을 보면 RSA public, private key 의 구성 요소들을 알 수 있다.
    
        RSAPublicKey ::= SEQUENCE {
          modulus           INTEGER,  -- n
          publicExponent    INTEGER   -- e
        }
        
        RSAPrivateKey ::= SEQUENCE {
          version           Version,
          modulus           INTEGER,  -- n
          publicExponent    INTEGER,  -- e
          privateExponent   INTEGER,  -- d
          prime1            INTEGER,  -- p
          prime2            INTEGER,  -- q
          exponent1         INTEGER,  -- d mod (p-1)
          exponent2         INTEGER,  -- d mod (q-1)
          coefficient       INTEGER,  -- (inverse of q) mod p
          otherPrimeInfos   OtherPrimeInfos OPTIONAL
        }
        

## JWT + JWK 실 사용 예시

AWS에서 제공하는 Cognito User Pool 인증을 통해서 발급받은 JWT를 이용하여 API 서버에서도 인증을 구현하고싶다면 어떻게 하면 될까?

흐름을 간단히 요약하면 다음과 같다.

  1. API 서버로 보내는 요청에 발급받은 JWT를 같이 전송
  2. API 서버에서 JWT의 유효성을 검증 (JWT발급자가 제공하는 URL에서 JWK 형식으로 된 public key를 다운로드 후 서명 일치 여부 확인)
  3. 검증 통과시 JWT에서 유저ID를 추출
  4. 해당 요청이 추출된 유저ID 사용자가 보낸 것임을 알 수 있음 (인증 완료)

위 프로세스를 진행하기 위해 AWS Cognito User Pool 기준으로 다시 필요한 내용을 정리해보자. [JWT의 유효성 검증(verify) 하기 위한 AWS 가이드][5] 에 나와있는 내용을 요약해보면 다음과 같다.

  * UserPool에서 발급한 JWT는 RS256 signature를 사용한다.
  * RS256 signature 검증 하기 
      * JWK URL에서 Public key키 가져오기 
          * AWS에서 제공하는 URL 규칙: https://cognito-idp.{region}.amazonaws.com/{userPoolId}/.well-known/jwks.json
      * 전달받은 JWT에서 SHA256(message)를 계산</p> 
      * 전달받은 JWT의 signature를 public key로 복호화 하여 위의 값과 동일한지 확인 (signature는 SHA256(message)를 private key로 암호화한 값이기 때문)

추가로, [Auth0에서 JWT, JWK를 활용하는 방식][6]도 참고 할만 하다.

 [1]: https://tools.ietf.org/html/rfc7519
 [2]: https://jwt.io/
 [3]: https://tools.ietf.org/html/rfc7517
 [4]: http://www.ietf.org/rfc/rfc3447.txt
 [5]: https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html
 [6]: https://auth0.com/blog/navigating-rs256-and-jwks/