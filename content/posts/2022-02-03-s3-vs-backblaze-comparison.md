---
title: Backblaze + 클라우드플레어를 이용한 서버 비용 절약 방법
date: 2022-02-02T15:15:59+00:00
url: /s3-vs-backblaze-comparison/
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/aws-application-load-balancer-custom-error-message/"     class="post-832"><span class="crp_title">AWS Application Load Balancer Custom Error Message</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/kotlin-coroutine-vs-javascript-async-comparison/"     class="post-873"><span class="crp_title">JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기</span></a></li><li><a href="https://www.letmecompile.com/eb-ec2-instance-graceful-shutdown/"     class="post-824"><span class="crp_title">Elastic Beanstalk 및 EC2 인스턴스 Graceful shutdown 설정</span></a></li><li><a href="https://www.letmecompile.com/%ea%b0%9c%eb%b0%9c%ec%9e%90%eb%a5%bc-%ec%9c%84%ed%95%9c-%ed%9a%a8%ec%9c%a8%ec%a0%81%ec%9d%b8-macos-%eb%b0%b1%ec%97%85-%eb%b0%a9%eb%b2%95/"     class="post-865"><span class="crp_title">개발자를 위한 효율적인 MacOS 백업 방법</span></a></li><li><a href="https://www.letmecompile.com/aws-application-load-balancer-custom-error-message/"     class="post-832"><span class="crp_title">AWS Application Load Balancer Custom Error Message</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps
  - AWS

---
<div class="wp-block-jetpack-markdown">
  <p>
    AWS S3 + 클라우드플레어를 사용하면 사용자와 클라우드플레어 CDN간의 대역폭 비용은 무료이지만 여전히 S3에서 클라우드플레어로 전송되는 대역폭에 대한 비용을 지불해야합니다. 즉, 클라우드플레어 CDN에서 캐시 미스(cache miss)가 많이 발생해서 오리진까지 요청이 많이 가면 오리진까지 도달한 해당 요청/응답에 사용된 대역폭 비용을 AWS에 지불해야 한다는 뜻 입니다. 반대로 말해 클라우드플레어에서 캐시 히트(cahce hit)가 되는 경우에는 S3까지 요청이 도달하지 않으니 S3 비용도 발생하지 않습니다.
  </p>
  
  <p>
    클라우드플레어 CDN에서 100% 캐시 히트가 될 수는 없으니 일부 요청은 오리진으로 도달할 것이고 이 요청이 많아질 수록 비용이 발생하게 되는데, 이러한 비용을 내지 않기 위해서 S3를 대체할만한 서비스들 중 클라우드플레어와 Bandwidth Alliance 로 맺어진 회사의 서비스를 사용하면 됩니다. 여기서는 대표적으로 Backblaze라는 서비스를 검토해 보겠습니다.
  </p>
  
  <h2>
    Backblaze vs. S3 비교 검토
  </h2>
  
  <p>
    결론부터 말하면 S3 정적 웹사이트 호스팅 기능을 사용하고 있는 경우 완전히 S3를 Backblaze로 대체하는것은 불가능합니다. 다만 용량이 큰 리소스를 서빙하기 위해서 Backblaze를 사용하면 비용을 많이 절감이 가능합니다.
  </p>
  
  <ul>
    <li>
      <p>
        redirection을 설정한다거나, https://letmecompile.com/my-page/ 로 접속하면 자동으로 https://letmecompile.com/my-page/index.html 를 찾아서 보여주는 등의 기본적인 웹서버의 동작을 지원하지 않음
      </p>
    </li>
    
    <li>
      <p>
        full path를 정확히 입력해야만 해당 파일에 접근이 가능하기 때문에 backblaze만으로는 완전한 하나의 정적 웹사이트를 호스팅하는 것이 불가능
      </p>
    </li>
    
    <li>
      <p>
        클라우드플레어에 CNAME을 등록하면 아래처럼 path부분을 그대로 유지하면서 host만 바꾸는 1:1 맵핑은 잘 작동함
      </p>
      
      <ul>
        <li>
          CNAME 적용전: https://f004.backblazeb2.com/file/bucket-name/profile.png
        </li>
        <li>
          CNAME 적용 후: https://image.letmecompile.com/file/bucket-name/profile.png
        </li>
      </ul>
    </li>
    
    <li>
      <p>
        클라우드플레어 워커를 같이 사용하면 이러한 단점 극복이 가능하지만 워커도 완전히 무료가 아니기 때문에 &#8216;S3 단독사용&#8217; vs &#8216;Backblaze + 클라우드플레어 워커&#8217; 비교했을 때 어느쪽이 더 비용 측면에서 유리한지는 상황에따라 다를 수 있음
      </p>
    </li>
    
    <li>
      <p>
        대신 특정 경로에 단순히 파일 업로드해놓고 이미지 서버 혹은 파일 서버로 활용하는 용도인 경우 Bandwidth Alliance덕분에 Backblaze를 사용하면 대역폭 비용을 완전히 무료로 사용가능 (Backblaze가 저장 용량 대비 비용 또한 S3 보다 훨씬 저렴)
      </p>
    </li>
    
    <li>
      <p>
        주의: 클라우드 플레어 SSL 설정시 전체구간(full) 옵션을 사용해야함. Backblaze의 웹서버는 https만 허용하기 때문에 flexible은 사용 불가. 반대로 S3 정적 웹사이트 호스팅의 경우 http만 지원하기 때문에 flexible만 사용 가능
      </p>
    </li>
    
    <li>
      <p>
        S3 호환 프로토콜을 사용하기때문에 기존에 AWS CLI를 잘 사용하고있었다면 매우 쉽게 마이그레이션 가능.
      </p>
      
      <ul>
        <li>
          <p>
            별도의 프로그램 설치 없이 aws s3 명령어를 그대로 사용하되, 다만 모든 명령어에 아래처럼 Backblaze 버킷에 해당하는 endpoint를 별도 지정해야함
          </p>
        </li>
        
        <li>
          <p>
            &#8211;endpoint-url=https://s3.us-west-004.backblazeb2.com
          </p>
        </li>
        
        <li>
          <p>
            예시)
          </p>
          
          <pre><code>    aws s3 sync \
    --endpoint-url=https://s3.us-west-004.backblazeb2.com \
    --acl public-read \
    --cache-control public,max-age=60,s-maxage=3600 \
    --exclude "*" \
    --include "*.html" --include "*.json" \
    --delete \
    ./build s3://bucket-name
</code></pre>
</li>
</ul>
</li>
</ul>

<h2>
  참고
</h2>

<ul>
  <li>
    https://help.backblaze.com/hc/en-us/articles/217666928-Creating-a-Vanity-URL-with-B2
  </li>
  <li>
    https://jross.me/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers/
  </li>
  <li>
    https://www.backblaze.com/blog/backblaze-and-cloudflare-partner-to-provide-free-data-transfer/
  </li>
</ul>
</div>