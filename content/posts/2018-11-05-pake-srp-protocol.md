---
title: PAKE와 SRP Protocol을 이용한 인증
date: 2018-11-04T17:13:01+00:00
url: /pake-srp-protocol/
dsq_thread_id:
  - 7019234276
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kerberos-protocol/"     class="post-737"><span class="crp_title">커버로스 프로토콜(Kerberos Protocol) - 서버 접근 권한 관리</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/kerberos-protocol/"     class="post-737"><span class="crp_title">커버로스 프로토콜(Kerberos Protocol) - 서버 접근 권한 관리</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Server

---
## PAKE (password authenticated key exchange)

[PAKE][1]는 두명 이상의 참여자가 패스워드 기반으로 암호화된 채널을 만들어서 서로 통신할 수 있게 해주는 암호학 적인 방법을 말한다.

  * Balanced PAKE 
      * 참여자들이 서로 암호화된 채널을 만드는 과정을 위해 동일한 패스워드를 사용하는 방법
  * Augmented PAKE 
      * server/client 시나리오에서 많이 사용된다.
      * 서버는 어떤 password-equivalent 데이터도 저장하지 않는다. (서버는 password로 부터 계산된 값을 보관하고 있지만 이 값으로는 원래 password를 역추론할 수 없다. 이 외에도 password를 역추론 해 낼 수 있는 관련 정보는 저장하지 않는다.)
      * 클라이언트가 서버에 인증을 시도할 때도 password를 역추론 해 낼 수 있는 직접적인 정보는 클라이언트에서 서버로 절대로 전송되지 않는다.
      * 때문에 공격자가 서버의 데이터를 접근 권한을 탈취했다고 하더라도 해당 데이터로 부터 password를 brute force로 찾아내기 전에는 client를 impersonate 하는 것이 불가능하다.

## SRP (Secure Remote Password) Protocol

[SRP 프로토콜][2]은 기존 특허를 피해서 만들어진 augmented PAKE의 한 종류이다. 인증 과정에서 password를 유추할 수 있는 직접적인 정보가 원격지로 전달되지 않아서 안전하다는 뜻으로 이름을 이해하면 될 것 같다. (이 이름이 지어진 정확한 기원은 찾지 못했다) SRP protocol은 여러번 수정되어왔고 현재는 revision 6a 버전이다.

  * 각 참여자가 인증을 위해 알고있어야 하는 정보 
      * 클라이언트 &#8211; 사용자의 패스워드(password)
      * 서버 &#8211; 사용자가 가입할때 입력한 ID, password로부터 유도된 cryptographic verifier, salt를 보관 
          * x = hash(salt, password)
          * verifier = g^x
          * 주의: x는 password를 추측해 낼 수있는 값이기 때문에 verifier를 만들고난 후 저장하지 않고 버린다.
  * 인증 흐름 (password를 직접 보내지 않고, 클라이언트가 password를 알고있다 라는 것을 증명하기 위한 절차) 
      * _주의: 아래 절차는 매 로그인 시도 마다 처음부터 반복된다._
      * 참고: <https://github.com/mozilla/node-srp> 에 더 실용적인 설명이 잘되어있다.
      * 클라이언트(a), 서버(b) 에서 각각 하나씩 랜덤 값 생성
      * 클라에서 UserID, A = g^a 를 서버로 전송하면 서버는 salt, B = kv + g^b 를 돌려줌 
          * k값은 클라/서버 모두 미리 알고있는 값
      * 현재까지 공유된 정보들과 원래 알고 있는 정보를 통해 서버/클라이언트에서 각각 공유된 세션 키 K를 계산해 낸다. (클라이언트에서 세션키 K를 계산해내는데 패스워드가 필요하다) 
          * 이 K를 사용하여 차후에 채널을 암호화 하는데 이용할 수있다.
      * 세션 키와 별도로 수학적 계산을 통해 M1을 계산 후 서버로 전송하면 M1을 서버에서 확인. (실제 이 단계에서 실제 인증이 완료된다) 
          * M1이 맞는 값임이 판별되면 클라에서 입력한 패스워드가 맞다는 의미 &#8211; 이때 서버/클라가 각자 계산했던 세션키 K는 동일한 값인 것을 확인 가능
          * M1이 틀리게 판별된 경우, 클라에서 입력한 패스워드가 틀렸다는 의미 &#8211; 이때 서버/클라가 각자 계산했던 세션키 K는 동일하지 않다.
  * 장점 
      * 인증 성공시 cryptographically-strong secret을 client/server 모두 획득하게된다. 이를 이용하여 참여자간 암호화된 통신이 가능. 
          * 인증 + 암호화된 통신 두가지 기능이 모두 필요한 경우에 매우 유용하고 SSH보다 더 안전함
          * SRP 레벨에서 암호화된 통신을 지원하므로 Trusted key servers와 certificate infrastructures (ex: SSL) 없이도 안전한 통신이 가능하고, 별도의 layer이기 때문에 두가지를 같이 사용하는것도 가능하다.
      * zero-knowledge password proof (ZKPP) 를 구현 
          * 한 참여자 (the prover, 보통 클라이언트)가 다른 참여자 (the verifier, 보통 서버)에게 “암호를 알고있다는 사실” 외에는 아무것도 밝히지 않고서도 자신이 암호를 알고있다는 것을 증명하는 방법
          * 어떤 참여자들도 다른 참여자 모르게 암호가 맞는지 확인 하는 것이 불가능하다. 즉, 암호가 맞는지 확인할 때마다 참여자들간에 한번의 인터랙션이 요구된다. 
              * 때문에 offline dictionary attacks 에 대해서도 안전하다
              * weak passwords를 사용하더라도 안전한 편
      * 독립적인 trusted third party 가 필요 없다 (Kerberos protocol의 경우 신뢰할 수있는 별도의 인증 서버가 필요함)
      * 기존 방식의 경우 MITM 공격에 의해 전송된 password가 한번 노출되면, 이를 replay해서 경우 공격자가 사용자를 지속적으로 가장(impersonate)하는 것이 가능했다. 
          * 하지만 SRP의 경우 특정 사용자에 대해 인증이 MITM에 의해 한번 노출되더라도 공격자가 해당 사용자를 지속적으로 impersonate할 수 없다. 패스워드를 직접적으로 유추할 수있는 정보가 전송되지 않으며, 그마저도 매 로그인 시도마다 클라이언트/서버간에 랜덤값 교환 절차가 포함되어있어서 값이 계속 변경되기 때문

## 실제 사용 예시

  * AWS Cognito의 User Pool에 대해 인증을 할때 InitiateAuth API의 AuthFlow를 USER\_SRP\_AUTH 로 설정할 경우 SRP protocol에 따라 로그인을 진행 할 수 있다. 
      * 클라에서 USERNAME, SRP\_A(A의 역할)를 Cognito server로 전송하면 Session과 PASSWORD\_VERIFIER(B의 역할)를 돌려받는다. 
      * 클라에서 PASSWORD_VERIFIER를 이용하여 최종적으로 M1을 계산 후, 해당값을 Session과 함께 RespondToAuthChallenge API에 넣어서 보내면 인증이 완료되고 OpenID Connect Token을 응답으로 돌려받는다.
      * 참고: 이 과정들은 AWS SDK에 의해 핸들링 되기때문에 직접 뭔가를 구현 할 필요는 없다.
      * <https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/amazon-cognito-user-pools-authentication-flow.html>
      * <https://docs.aws.amazon.com/cognito-user-identity-pools/latest/APIReference/API_InitiateAuth.html>
      * <https://docs.aws.amazon.com/cognito-user-identity-pools/latest/APIReference/API_RespondToAuthChallenge.html>

 [1]: https://en.wikipedia.org/wiki/Password-authenticated_key_agreement
 [2]: https://en.wikipedia.org/wiki/Secure_Remote_Password_protocol