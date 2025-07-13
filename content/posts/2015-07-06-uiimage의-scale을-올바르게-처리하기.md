---
title: UIImage의 scale을 올바르게 처리하기
date: 2015-07-06T02:21:33+00:00
url: /uiimage의-scale을-올바르게-처리하기/
dsq_thread_id:
  - 3908006627
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - iOS

---
`UIImage`의 size 는 픽셀(px)단위가 아닌 포인트(point) 단위로 처리가된다. 따라서 사이즈 정보에 추가적으로 스케일(scale) 정보를 같이 같고있다. `UIImage`를 bundle에서 `[UIImage imageNamed:]`를 이용하여 불러올때는 이미지에 붙어있는 `@2x` 또는 `@3x` 포스트픽스를 통해서 해당이미지의 스케일이 정상적으로 설정되어 특별히 신경쓸 필요가 없다. 마찬가지로 서버에 이미지 데이터를 요청해서 받아올 경우에도 AFNetworking의 `AFImageResponseSerializer`을 사용하면 `UIImage`를 만들어줄 때 스케일 관련 정보 또한 내부적으로 처리를 해주니 신경을 쓸 필요가 없다.

하지만 `NSData`로부터 `UIImage` 를 생성할때나 파일명에 포스트 픽스가 없는 다운로드된 파일로 부터 `UIImage`를 생성할 때는 이러한 스케일 처리를 따로 해줘야 된다.

다음과 같이 처리해주면 현재 디바이스의 스크린 스케일을 이용해서 디바이스에 맞게 이미지를 불러올 수 있다.

    + (UIImage *)category_imageWithData:(NSData*)data
    {
        UIImage *image = [[UIImage alloc] initWithData:data];
        return [[UIImage alloc] initWithCGImage:[image CGImage] scale:[UIScreen mainScreen].scale orientation:image.imageOrientation];
    }
    

위와같이 스케일을 제대로 지정해주지 않을경우 어떤경우에 문제가 되는지 다음을 살펴보도록 하자.

## 문제 상황 분석

  * NSData or postfix가 없는 파일로부터 UIImage 만들어낼때 scale을 따로 지정하지않으면 scale = 1.0 이다.
  * 이렇게 만들어진 이미지는 픽셀기준이므로, 디바이스에따라 포인트 기준으로 resize 동작 등 이미지 관련 조작을 할때 기대와 다르게 실행될 수 있다.

### 문제 예시

  * postfix가 없는 파일 이미지데이터의 실제 사이즈가 600x600px이라고 가정.
  * `UIImageView` 의 사이즈가 100&#215;100 이라 해당 이미지를 리사이즈해서 사용하기 위해 리사이즈 메서드 `[myImage category_myCustomResize:CGSizeMake(100,100)];` 을 실행 
        - (UIImage*)category_myCustomResize:(CGSize)size 
        {
            UIGraphicsBeginImageContextWithOptions(size, NO, self.scale);
            [self drawInRect:CGRectMake(0, 0, size.width, size.height)];
            UIImage* image = UIGraphicsGetImageFromCurrentImageContext();
            UIGraphicsEndImageContext();
        
            return image;
        }
        

  * 스케일 반영되어 포인트기준으로 생성된 myImage (width:300, scale:2) -> 리사이즈시 width:100, scale:2(실제 200px image 생성됨: 원하는 결과)

  * 1x 스케일로 픽셀기준으로 생성된 myImage (width: 600, scale:1) -> 리사이즈시 width:100, scale:1(실제 100px image 생성됨: 문제발생! 잘못된 결과)