---
title: '인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format & extensions)'
date: 2018-10-19T09:02:10+00:00
url: /certificate-file-format-extensions-comparison/
dsq_thread_id:
  - 6980602868
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 웹 개발

---
https를 지원하는 웹서버를 설정하거나 서명이나 암호화 관련된 개발을 하게되면 한번씩 인증서 관련된 파일을 다룰 일이 생기게 된다. 이때 항상 프로그램이나 라이브러리들이 지원하는 형식이 달라서 인증서 형식을 변환해아 하는데 현재 갖고있는 파일의 형식이 무엇인지를 알아야 제대로 활용이 가능하다. 인증서 파일의 경우 인코딩 방식과 확장자가 일치하는 경우도 있고, 그렇지 않은 경우도 있기 때문에 아래와 같이 비교해서 정리해 보았다.

  * 인코딩 (확장자로 쓰이기도 한다.) 
      * `.der`: 
          * Distinguished Encoding Representation (DER)
          * 바이너리 DER 형식으로 인코딩된 인증서. 텍스트 편집기에서 열었는데 읽어들일 수 없다면 이 인코딩일 확률이 높다.
      * `.pem`: X.509 v3 파일의 한 형태 
          * PEM (Privacy Enhanced Mail)은 Base64인코딩된 ASCII text file이다.
          * 원래는 secure email에 사용되는 인코딩 포멧이었는데 더이상 email쪽에서는 잘 쓰이지 않고 인증서 또는 키값을 저장하는데 많이 사용된다.
          * `-----BEGIN XXX-----`, `-----END XXX-----` 로 묶여있는 text file을 보면 이 형식으로 인코딩 되어있다고 생각하면 된다. (담고있는 내용이 무엇인지에 따라 XXX 위치에 CERTIFICATE, RSA PRIVATE KEY 등의 키워드가 들어있다)
          * 인증서(Certificate = public key), 비밀키(private key), 인증서 발급 요청을 위해 생성하는 CSR (certificate signing request) 등을 저장하는데 사용된다.
  * 확장자 
      * `.crt`, `.cer` 인증서를 나타내는 확장자인 cer과 crt는 거의 동일하다고 생각하면 된다. (cer은 Microsoft 제품군에서 많이 사용되고, crt는 unix, linux 계열에서 많이 사용된다.) 
          * 확장자인 cer이나 crt만 가지고는 파일을 열어보기 전에 인코딩이 어떻게되어있는지 판단하긴 힘들다.
      * `.key`: 개인 또는 공개 PKCS#8 키를 의미
      * `.p12`: PKCS#12 형식으로 하나 또는 그이상의 certificate(public)과 그에 대응하는 private key를 포함하고 있는 key store 파일이며 패스워드로 암호화 되어있다. 열어서 내용을 확인하려면 패스워드가 필요하다.
      * `.pfx`: PKCS#12는 Microsoft의 PFX파일을 계승하여 만들어진 포멧이라 pfx와 p12를 구분없이 동일하게 사용하기도 한다.

## 표준 비교 PKCS#8, PKCS #12, X.509

  * PKCS #8은 Public-Key Cryptography Standards (PKCS) 표준 중의 일부로 private key를 저장하는 문법에 관한 표준이다. PKCS #8 private keys 는 일반적으로 PEM 형식으로 인코딩된다.
  * PKCS #12는 하나의 파일에 여러 암호화 관련 엔티티 들을 합쳐서 보관하는 방식에 관한 표준이다.
  * X.509는 public key infrastructure (PKI)에 대한 ITU-T의 표준 (RFC 5280) 으로 공개키(public key) 인증서의 format, revocation list(더이상 유효하지 않은 인증서들에 대한 정보 배포), certification path(chain) validation 알고리즘 등을 정의한다. 보통 X.509 인증서라 하면 RFC 5280에 따라서 인코딩되거나 서명된 디지털 문서를 말한다.</p> 
  * 참고: PEM 방식으로 인코딩된 pkcs8 형식(format) 비밀키(private key)를 암호화되지 않은 DER 인코딩으로 변경하는 명령어
    
        openssl pkcs8 -topk8 -inform PEM -outform DER -in private.key -out private.der -nocrypt
        

## 암호화/복호화 PKCS#1Padding, PKCS#5Padding, PKCS#7Padding의 차이점

암호화 복호화 코드를 작성해본 사람이라면 분명히 Padding과 관련된 코드를 한번이라도 보았을 것이다. 이름이 전혀 직관적이지 않다보니 처음 접했을 때는 도데체 뭘 의미하는지 알기가 어려웠지만 하나씩 차근차근 알아보도록 하자.

#### PKCS5Padding, PKCS7Padding

대칭키 알고리즘(ex: AES)을 사용 할 때 암호화 하려는 데이터의 길이가 긴 경우 ECB (Electronic Codebook), CBC(Cipher Block Chain) 등의 block cipher 방식을를 사용해서 데이터를 암호화 하게된다. 블록단위로 잘라서 암호화/복호화가 진행되기 때문에 정해진 마지막 블록의 사이즈와 실제 데이터의 크기가 맞지 않는 경우가 생길 수 있다. 이때 마지막 블록의 빈 공간을 채워넣는(padding) 방식을 말한다.

예를 들어 데이터가 17바이트인데 block size가 8바이트라면 총 3개의 블록이 필요하고 마지막 블록은 7바이트가 남기때문에 이부분을 특정 값으로 채워넣어서 암호화를 진행한다.

PKCS5Padding과 PKCS7Padding 둘다 빈 공간에 값을 채워넣는 규칙은 다음과 같다.

  * 마지막 블록에 1바이트가 남는다면 `01`을 채워넣고 (`01`은 1바이트를 16진수 표기한 값)
  * 마지막 블록에 2바이트가 남는다면 `02 02`를 채워넣음
  * 마지막 블록에 3바이트가 남는다면 `03 03 03`를 채워넣음 

대신 PKCS5Padding과 PKCS7Padding의 경우 block size의 제약 조건이 다르다.

  * PKCS5Padding의 경우 block size 8바이트로 고정
  * PKCS7Padding의 경우 block size 최대 255바이트까지 지원

#### PKCS1Padding

PKCS1Padding는 앞서말한 PKCS5Padding/PKCS7Padding과는 방식도 다르고 사용처도 다르다. PKCS1Padding는 보통 RSA 알고리즘과 같이 사용되며 [PKCS#1 v1.5][1] 에 명시된 padding 방법을 사용한다.

  * Encryption Block 구성 = 00 || Block Type || Padding || 00 || Data
  * 최소 11바이트의 패딩이 추가된다. (오버헤드가 크다) 이 중 8바이트는 1~255 사이의 랜덤 값으로 채워진다.
  * 이런 랜덤 특성 때문에 동일한 데이터를 암호화 해도 매번 다른 결과가 나온다.
  * RSA의 특성상 보통 content전체를 암호화하는데 사용되지 않고, content를 암호화 하는데 사용한 content key(private key)를 암호화 하기 위한 수단으로 사용되기 때문에 메세지의 크기가 늘어나는 패딩 오버헤드는 크게 문제되지 않는다. 

#### Crpyto Library in Java

Java에서 제공하는 Crypto 라이브러리의 경우 Cipher.getInstance(&#8220;algorithm/mode/padding&#8221;) 형식의 스트링으로 암호화 알고리즘, 모드, 패딩방식을 결정할 수 있다. 참고로 데이터의 길이가 길어서 (블럭의 갯수가 하나 이상)인 경우 ECB는 보안성이 취약하니 CBC를 꼭 사용해야 한다. (각 블럭의 데이터가 섞이지 않은 상태로 동일한 암호화를 적용하면 데이터에 패턴이 나타나게됨)

예시)

  * &#8220;RSA/ECB/PKCS1Padding&#8221; (참고: 이 방식은 1개의 블록만 암호화 할 수 있도록 구현되어있기 때문에 실제로는 ECB라고 부를 수 없다.)
  * &#8220;AES/ECB/PKCS5Padding &#8220;
  * &#8220;AES/CBC/PKCS5Padding &#8220;
  * &#8220;AES/ECB/PKCS7Padding &#8220;
  * &#8220;AES/CBC/PKCS7Padding &#8221; 

## OpenSSL을 이용하여 인증서 조작하기

인증서의 정보를 보거나(View), 다른 포멧으로 변환(Transform), 합침(Combination) 등의 조작을 하기위해서 openSSL 명령어를 사용할 수 있다.

(openssl 명령어의 경우 inform 옵션이 따로 지정되지 않은경우 PEM형식으로 처리를 시도한다.)

  * View 
      * PEM: openssl x509 -in cert.pem -text -noout
      * DER: openssl x509 -in cert.der -inform der -text -noout
      * 파일의 인코딩 형식을 미리 알고있지 않은경우 각 명령어를 한번씩 실행해보고 정상적으로 처리되는지 확인
  * Transform 
      * PEM -> DER: 
          * openssl x509 -in cert.crt -outform der -out cert.der
      * DER -> PEM: 
          * openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
  * Combination 
      * PEM 의 경우 여러 파일들을 하나로 합치는 것도 가능하다. text 파일이기때문에 잘 복사해서 한파일에 같이 붙여넣고 사용하면된다. 
      * 예를들어 Apache SSL 설정에서 인증서 체인을 구성할때 Server cert -> Intermediate cert -> Root cert 순으로 붙여 넣고 저장한 후 Apache에서 해당 파일을 설정해주면 된다.

## 부록: RSA 키 페어 생성 하기

외부 인증기관에서 인증서를 전달받지 않고, 내부적으로 사용할 RSA 키 페어가 필요하다면 아래 설명된 절차를 통해서 간단하게 키 페어를 만들어 낼 수 있다.

#### RSA key pair 생성

    openssl genrsa -des3 -out private.pem 2048
    

private.pem파일을 열어보면 `-----BEGIN RSA PRIVATE KEY-----` 로 표시되는 것을 확인 할 수 있다.

#### Private key에 포함된 정보 확인

RSA private key로부터 public key를 만들어 낼 수 있다. 어떻게 그게 가능한지 보기 위해서 아래 명령어를 사용해보자.

    openssl rsa -text -in private.pem 
    

위 명령어로 출력된 결과에 public key를 정의하는데 필요한 modulus와 publicExponent정보가 포함되어있는것을 알 수있다. 실제로 private key로부터 public key를 추출하기 위해서 다음 단계를 살펴보자.

#### Private key에서 Public key추출하기

    openssl rsa -in private.pem -outform PEM -pubout -out public.pem
    

public.pem 파일을 열어서 `-----BEGIN PUBLIC KEY-----` 로 표시되는것을 확인 하면 된다.

## References

  * [RFC 5280: Internet X.509 Public Key Infrastructure Certificate  
    and Certificate Revocation List (CRL) Profile][2]
  * [PKCS #1: RSA Encryption Version 1.5][1]
  * [PKCS #7: Cryptographic Message Syntax][3]
  * [PKCS #8: Private-Key Information Syntax Specification][4]
  * [PKCS #12: Personal Information Exchange Syntax Standard][5]
  * [der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them][6]
  * [OpenSSL PKCS8][7]
  * [OpenSSL RSA][8]
  * [using-ecb-as-rsa-encryption-mode][9]

 [1]: https://tools.ietf.org/html/rfc2313
 [2]: https://tools.ietf.org/html/rfc5280
 [3]: https://tools.ietf.org/html/rfc2315
 [4]: https://tools.ietf.org/html/rfc5208
 [5]: https://tools.ietf.org/html/rfc7292
 [6]: https://support.ssl.com/Knowledgebase/Article/View/19/0/der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them
 [7]: https://www.openssl.org/docs/manmaster/man1/pkcs8.html
 [8]: https://www.openssl.org/docs/manmaster/man1/rsa.html
 [9]: https://crypto.stackexchange.com/questions/25899/using-ecb-as-rsa-encryption-mode-when-encrypted-messages-are-unique