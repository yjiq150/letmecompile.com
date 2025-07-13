---
title: iOS 인앱 정기결제(IAP Auto-renewable Subscription) 튜토리얼
date: 2016-06-26T15:10:33+00:00
url: /in-app-purchase-auto-renewable-subscription/
dsq_thread_id:
  - 4940097540
dsq_needs_sync:
  - 1
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-auto-increment-%ec%84%b1%eb%8a%a5-%ec%b5%9c%ec%a0%81%ed%99%94/"     class="post-750"><span class="crp_title">MySQL - InnoDB Auto Increment 성능 최적화</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - iOS

---
iOS 앱에서 상품을 등록하고 판매하는 과정은 꽤나 복잡하다. 그 중에서도 정기구독 자동결제(Auto-Renewable Subscription) 상품을 판매하는 경우 신경써야 할 부분이 매우 많다. 2016년 WWDC에서 애플은 Auto-Renewable Subscription을 모든 카테고리의 앱에 적용가능하도록 허용하기로 했고(기존에는 잡지, 음악 등 특정 컨텐츠에 대해서만 허용되었었음), 해당 타입의 결제를 통해 발생한 매출의 경우 다음 조건을 만족할 경우 앱 판매 수수료를 30% ->15%로 인하 하는 내용에 대해서 [발표][1]했다.

  * 수수료 인하 조건 
      * 해당유저가 1년이상 결제를 유지한 경우 1년 이후 결제분의 수수료를 15%로 인하 (2016년 6월 13일 이후 부터 적용) 
      * 사용자가 중간에 정기 결제를 취소했더라도 60일 이내에 다시 정기결제를 시작한경우 해당 유저의 결제 기간은 계속 누적된다. (60일을 넘을 경우 리셋)
      * 이번 발표 전 기존 결제 기간도 카운트 된다. (Prior days of paid service are counted.) 즉, 이미 1년간 결제해온 사용자가 6월 13일 이후에 결제할 경우 수수료는 15%로 줄어든다.

Auto-Renewable Subscription에 대해 자세히 설명하기 전에 먼저 앱스토어의 다른 상품 타입들에 대해서도 간략히 정리해보도록 하자.

## 인앱 결제 (In-App Purchase) 종류

  * Consumable products: 결제 후 사용하면 일회성으로 소비되는 상품. 예를 들어 voice talk credit이나 게임상에서 사는 게임 머니 등이 있다. 
  * Non-consumable products: 한 번 사면 무한정 가지고 있을 수 있는 상품. unlock한 게임 레벨, 책 컨텐츠 , 카메라 앱에서 구매한 스티커나 프레임등이 여기에 해당된다. 
  * Auto-renewable subscription: 정기적으로 결제가 일어나는 상품으로 일정 기간 동안만 컨텐츠 이용이 가능하다. active한 기간동안 모든 device에서 접근 가능해야 하며 이는 restore 가 되어야 함을 뜻한다. non-consumable과는 다르게 expiration date가 존재하며 만료 전 ios 시스템에서 자동으로 renewal을 시도한다. 음악 정기결제, 잡지 정기 결제 등이 해당된다. 
  * Non-renewable subscriptions: 일정 기간 동안만 사용 가능한 상품으로 자동갱신이 일어나지 않는다. 

## 인앱 결제 구현하기

Auto-Renewable Subscription 상품에 대한 결제 방식은 다른 종류의 IAP 상품들과 동일하니 여기서 자세히 설명하지는 않고 넘어가도록 한다.

## 인앱 정기 결제시 리뉴얼 결제 절차

정기결제가 진행중인 경우 다음과 같은 절차를 통해서 리뉴얼 결제가 발생한다. <sup id="fnref-560-apple_subs"><a href="#fn-560-apple_subs" class="jetpack-footnote">1</a></sup>

  * “preflight” check &#8211; 만료 10일전에 애플이 알아서 점검 후 아래 항목에 해당하는 문제가 있을경우 유저에게 알림이 전달된다. 
      * 고객의 등록된 신용카드 정보가 유효한지
      * 고객이 첫 정기결제를 한 시점 이래로 가격이 변동되지 않았는지
      * 해당 프로덕트가 삭제되지 않았는지
  * 만료 24시간 전 
      * 애플이 자동으로 renew 를 위한 결제 시도를 진행. 너무 많이 실패시 시도 중단.
      * 앱의 기동 여부와 관계 없이 진행된다. 
      * 앱스토어는 lapse를 피하기 위해서 만료시간보다 살짝 일찍 결제 진행됨
      * 그래도 lapse가 생기는 경우 발생 
          * 유저의 결제정보가 올바르지 않은경우 리뉴얼결제실패 → 만료기간이 지난후에 다시 정상적인 결제정보가 입력된경우 lapse 발생 
              * 유저가 정기결제를 꺼놓았다가 만료기간이 지난후 다시 정기결제 켠 경우 lapse 발생
  * 리뉴얼 결제가 이루어졌는지 내역 확인 
      * 애플에서 발생한 리뉴얼 결제 여부에 대해서 서버나 앱에서 조회는 할 수 있으나 결제시점에 애플로부터 따로 notification을 받을 수 있는 방법은 없다. 
      * 따라서 앱의 기동여부와 관계없이 리뉴얼 결제여부를 알아내려면 서버에서 항상 polling을 통해 조회해야한다. 이경우 서버에서 누락되거나 처리가 delay 되는 경우가 발생할 수도있기 때문에 앱 기동시에도 한번더 리뉴얼 결제 여부를 조회하여 처리해줄 필요가 있다. (결국 앱기동시 앱에서 trigger하는 로직, 서버에서 polling으로 trigger하는 로직 둘다 필요함.)
      * 앱에서 확인 하기 
          * `[[SKPaymentQueue defaultQueue] addTransactionObserver:];` 를 앱 기동시에 실행하면, 리뉴얼 결제가 발생한 경우 paymentQueue에 결제건이 들어오게된다. 이 트랜잭션을 사용자가 직접 결제 했을때와 동일한 로직으로 처리하면 된다. 
          * 일반적으로 앱에서 locally 영수증 유효성을 검사하는것 보다 서버에서 애플서버로 요청을 날려 영수증 유효성을 조회하는것이 더 안전하다.
          * 따라서 결제 내용과 영수증을 자체 서버로 보낸 후, 자체 서버에서 애플서버로 다시한번 verfyReceipt를 하는 절차로 구성하면 된다. (아래 서버에서 확인 파트와 동일하게 처리)
      * Polling으로 서버에서 확인 하기 
          * 서버에서 주기적으로 batch작업을 통해 만료일이 가까워진 결제건에대해 애플서버에 아래와같이 polling을 해야한다.
          * 자체 서버에서 보관중인 original 결제건의 receipt를 이용하여 아래 URL에 조회를 해야한다. 
              * sandbox: https://sandbox.itunes.apple.com/verifyReceipt
              * production: https://buy.itunes.apple.com/verifyReceipt
          * 해당 URL 조회시 마지막 receipt 정보 + latest receipt 정보가 같이 리턴되서 돌아온다. 두가지가 다를경우 정상적으로 리뉴얼이 된것으로 보면 된다. 돌아온 receipt안에 정기결제 만료일, 취소일자 등의 주요 정보들이 있으니 항상 이 정보를 기준으로 모든 작업들을 처리하면 된다.
      * Server notification을 받아서 서버에서 확인하기 
          * polling의 비효율성을 해결해기 위해 애플에서도 [Server notification][2] 기능이 추가된 줄 알았으나.. notification을 발송해 주는 상황이 매우 제한적이라 큰 도움이 안된다(결국 앱기동시 리뉴얼 결제 처리 + 서버 polling 해야한다)

## 영수증 검증하기 (Receipt Validation) <sup id="fnref-560-apple_receipt"><a href="#fn-560-apple_receipt" class="jetpack-footnote">2</a></sup>

  * Receipt 
      * [[NSBundle mainBundle] appStoreReceiptURL] 주소를 읽어들이면 receipt을 NSData 형태로 읽어들일 수 있다.
      * 암호화 되어 저장되어있으며, 이 receipt data안에 모든 결제건(transactions)들이 다 들어있다. 새로운 결제건이 생길때마다 다시 업데이트 된다.
      * [<img loading="lazy" src="/uploads/2016/06/apple_app_receipt-819x1024.png" alt="" width="625" height="781" class="alignnone size-large wp-image-673" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2016/06/apple_app_receipt-819x1024.png 819w, https://www.letmecompile.com/wp/wp-content/uploads/2016/06/apple_app_receipt-240x300.png 240w, https://www.letmecompile.com/wp/wp-content/uploads/2016/06/apple_app_receipt-768x960.png 768w, https://www.letmecompile.com/wp/wp-content/uploads/2016/06/apple_app_receipt-150x188.png 150w, https://www.letmecompile.com/wp/wp-content/uploads/2016/06/apple_app_receipt.png 1192w" sizes="(max-width: 625px) 100vw, 625px" />][3]
  * 어떻게 검증하는 것이 안전한가? 
      * 앱에서 직접 receipt validation을 구현하는 것도 가능하지만 추천하진 않는다. 
          * https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateLocally.html
      * 서비스 서버로 읽어들인 receipt을 업로드 한 후, 서비스 서버가 애플서버의 verifyReceipt API를 호출해서 검증하는 것이 가장 secure한 방법이다. 
          * https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html
      * 서비스 서버를 거치지 않고 앱에서 직접 애플 서버와 통신하는 방식은 보안상 취약점이 존재하기때문에 추천하지 않는다.
  * 언제 영수증이 업데이트 되는가? 
      * 앱 기동시 SKPaymentQueue에 observer를 걸어두면 앱 기동시마다 receipt정보를 최신으로 유지할 수 있다.
      * 원래 사용중이던 A디바이스외에 추가로 다른 B 디바이스에서 결제가 일어났더라도 A디바이스에서 앱 을 기동시 앱스토어에서 receipt를 받아오게되고 새로 업데이트된 트랜잭션들이해당 receipt 안에 들어있다.
      * Auto-renewal의 경우 사용자가 앱을 사용하지 않고있는 중에도 결제가 일어날 수 있다. 첫 결제시에 서비스 서버에 receipt를 보관해두었으면, 서비스 서버가 해당 receipt를 이용하여 애플서버의 verifyReceipt를 호출하여 (polling 방식) 리뉴얼 결제가 일어났는지를 확인 할 수 있다. 
          * 결제가 일어났는지 서버에서 미리 확인이 완료되더라도 나중에 앱 기동시 updateTransactions() -> finishTransaction() 플로우를 동일하게 처리 해줘야 한다.
  * Auto-renewal 결제를 했던 유저가 subscription 중인지 어떻게 확인하는가? 
      * 첫 auto-renewal 결제의 transactionId가 originalTransactionId가 된다.
      * 이후 리뉴얼 결제들도 매 결제 건 마다 transctionId가 새로 부여되지만 originalTransactionId는 동일하다.
      * 따라서 originalTransctionId로 필터링해서 모든 결제건을 살펴보고 그중에 expireDate가 아직 미래의 날짜로 되어있는 결제건이 있다면 해당 유저는 subscription 중이라고 판단한다.
  * Enhanced Receipts &#8211; 2017년 7월 18일부로 기존에 비해 구독유저 정보 확인에 도움이 되는 더 자세한 정보가 추가되었다. 
      * 다음 결제일에 오토리뉴얼이 일어날 예정인지 확인 가능
      * 구독 가격을 인상한 경우 유저가 가격 조정에 동의 했는지 상태 
      * 리뉴얼 결제가 실패한경우 앱스토어가 해당 결제를 다시 재시도 하고있는지 여부
      * 구독 만료 이유(Expiration Intent) 
          * 구독 가격 인상시 조정 동의하지 않음
          * 신용카드 유효 기간 만료로 인한 결제 실패 등
      * 취소 이유 (Cancellation Reason) 
          * 애플 고객센터에 의한 취소 등

## 인앱 결제 내역 복원하기 (Restore In-App Purchase Products)

[<img loading="lazy" src="/uploads/2016/06/iap-types.png" alt="iap-types" width="511" height="175" class="alignnone size-full wp-image-562" />][4]

위 표에서 많이들 헷갈려 하는 부분이 Restored _by the system_ 과 _by your app_의 차이이다. by the system 이란 Apple에서 지원하는 StoreKit을 이용해서 Apple server와 통신해서 상품을 restore를 하는 것이고 by your app이란 Apple과의 통신 없이 서비스 서버에서 상품을 restore 해 주는 것이다.

  * Consumable Item 은 Restore의 대상이 아니다. 게임에서 사용하는 루비 같은 존재이므로 한번 구매하고 쓰지 못했더라도 다른 Device에서 복구해 줄 의무는 없다. 
  * Non-consumable / Auto-Renewable Subscription은 Apple에서 Restore를 지원한다. 자동으로 되는 건 아니고 Application내에 shop 페이지 어딘가에 Restore 버튼을 놓고 Store Kit 을 이용해서 개발해야 한다. Restore 버튼이 없으면 앱스토어 리뷰 과정에서 리젝(reject) 사유가 될 수 있다. 
  * Non-renewable Subscription은 Restore을 해줘야 하는 상품이지만 정작 apple server에서는 restore 지원을 해주지 않는다. 따라서 자체 서비스 서버를 통해 인앱 결제시에 사용자를 식별할 수 있는 정보를 저장하여 restore 기능을 구현해야한다. (restore를 지원한다기 보다는 계정에서 구매 정보를 유지해 주면 되는 형태이다.) [애플 도큐먼트][5]의 아래 글을 보면 명확하다.  
    > It’s your app’s responsibility to make the subscription available on all of the user’s devices and to let users restore the purchase. This product type is often used _when your users already have an account on your server that you can use to identify them when restoring content._ 

얼핏 보면 Non-renewable Subscription 은 로그인 한 유저만 살 수있도록 해도 무방할 것 같다. 그러나 stack over flow에서 Non-renewable Subscription를 구매할 때 Registration을 requirement로 했을 때 스토어 리뷰과정에서 reject 당했다는 사람이 많았다 ([참고][6]).

앱스토어에서 리젝을 피하기 위해 다음과 같은 가이드라인을 따르면 된다.

  * Non-renewable Subscription 을 구매할 때 registration 과정은 optional 이어야 한다. (즉, 로그인 하지 않은 user도 구매가능 하도록 허용해야한다.)
  * 다만 로그인 하지 않은 유저는 다른 디바이스에서 동일 앱을 사용 할 때나, 앱을 지웠다 다시 깔았을 때 해당 유저와 결제 내역을 연결할 정보가 없어서 restore가 불가능하다. 따라서 Non-renewable Subscription 상품을 구매하려고 할 때 이를 명시해서 “로그인을 하지 않으면 다른 device에서 restore 할 수 없습니다. 구매하시겠습니까?” 등의 메세지를 보여준다. 
  * 로그인 한 유저가 구매한 Non-renewable Subscription은 별도의 restore 과정없이 어떤 device에서 로그인 해도 보존 되어야 한다. 

## Code in Action: 인앱 결제 내역 복원하기

Apple Store Kit이 제공하는 Restore 기능은 해당 기기에 로그인된 Apple ID를 기준으로 동작한다. 이는 서비스 registration id 와는 상관이없다. 따라서 같은 Apple ID를 공유하면서 생길 수 있는 abusing 이슈에 대해서는 서비스 서버에서 따로 방어햐야 한다.

Restore 로직을 짜기 위해서 [SKPaymentTransaction][7] class 에 대한 이해가 필요하다.  
SKPaymentTransaction 는 상품 구매의 한 단위이다. 이는 한번 구매가 완료 되었을 때 (실패, 성공 상관없이) 하나 생기고 Restore 할 때도 하나가 추가로 생긴다. Restore를 10번 하게되면 10개가 더 생기게 된다.

  1. SKPaymentTransactionObserver protocol을 따르는 핸들러를 만든 후 restore transaction에 대한 이벤트를 받는다.  
    `@interface RestoreViewController : UIViewController <SKPaymentTransactionObserver>`
  2. transaction observer 를 등록한다. 모든 작업이 끝난 후 remove 해 주는것을 잊지말자. 
        [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
        //...
        [[SKPaymentQueue defaultQueue] removeTransactionObserver:self];
        

  3. restore button을 유저가 클릭했을 때 아래 함수를 부른다. 
        [[SKPaymentQueue defaultQueue] restoreCompletedTransactions];
        

  4. SKPaymentTransactionObserver에 대한 call back 함수를 작성해 준다. updatedTransactions: 함수는 required 이다. SKPaymentQueue 에 SKTransaction 이 담겨서 들어오는데 SKPaymentTransactionState 를 보고 transaction을 finish 해 주어야 한다. 이 함수는 observer가 잘 걸려있고 restoreCompletedTransactions 를 호출하면 잇달아 자동으로 불린다. 
        - (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions 
        {
            for (SKPaymentTransaction *aTransaction in transactions)
            {
                if (aTransaction.transactionState ==  SKPaymentTransactionStateRestored)
                {
                    [[SKPaymentQueue defaultQueue] finishTransaction:aTransaction]; 
                }
            }
        }
        

  5. restore 실패 시 아래 함수가 불리게된다. 
        - (void)paymentQueue:(SKPaymentQueue *)queue restoreCompletedTransactionsFailedWithError:(NSError *)error
        

  6. restore 성공시 paymentQueueRestoreCompletedTransactionsFinished: 함수가 불린다. 여기서 주의 할 점은 restore 된 transaction 또한 새롭게 생성된 transaction이기 때문에 transactionIdentifier 가 크게 의미가 없다는 것이다. 첫 결제의 transaction id는 originalTransaction 라는 property에서 참조하고 있다. 이 originalTransaction 의 transaction id가 중요하다.  
    > “originalTransaction : The contents of this property are undefined except when `transactionState` is set to `SKPaymentTransactionStateRestored`.” 

위 API문서에는 restored 된 transaction 은 항상 originalTransaction 을 가지고 있는 것 처럼 설명되어 있지만, Apple 버그인지는 몰라도 originalTransaction 이 nil로 넘어올 때가 있으니 꼭_ nil check를 하는 것이 중요하다. _

이렇게 해당 애플 아이디로 구매했던 non-consumable 과 auto-renewable subscription의 목록을 가져와서 서비스 서버로 응답해 주면 서버가 실제 그 user 에게 이전에 구매했던 상품들을 다시 matching 시켜주면 된다.

### 예제코드

    - (void)paymentQueueRestoreCompletedTransactionsFinished:(SKPaymentQueue *)queue
    {
        NSMutableSet* transactionIdList = [NSMutableSet new];
        for (SKPaymentTransaction *aTransaction in queue) 
        {
            if (aTransaction.transactionState ==  SKPaymentTransactionStateRestored)
            {
                if (aTransaction.originalTransaction) {
                    [transactionIdList addObject:aTransaction.originalTransaction.transactionIdentifier];
                }
    
                [[SKPaymentQueue defaultQueue] finishTransaction:aTransaction];
            }
        }
    
        // do something with transactionIdList 
    }
    

## 애플 샌드박스 계정을 통한 결제 테스트 (Test Using Sandbox)

이제 Auto-Renewable Subscription 상품에 대해 어느정도 개발이 진행되었으니 테스트를 해보도록 하자. 그런데 내가 설정한 상품의 subscription 기간이 1달이라면 다음 정기결제가 일어나는 시점까지 기다려서 테스트 하기가 매우 어려울 것이다. 아직 개발중인 앱의 경우 해당 상품이 앱스토어 상품 리뷰를 통과하지 않았기 때문에 경우 실제 Apple ID로 결제 테스트 하는 것도 불가능 하다. 이때 사용하는 것이 SandBox 계정이다. SandBox 계정은 Apple Developer Center에 가서 추가할 수 있다.

SandBox 결제의 특징들과 몇가지 테스팅 팁을 정리해보았다.

  * Renewal Time : SandBox를 이용하면 실제 리뉴얼 주기보다 훨씬 빠르게 renewal이 일어난다. 그리고 결제 transaction은 실제 리뉴얼 주기보다 조금 더 미리 일어난다. renewal은 최대 6회까지 일어난다. 대부분 6회까지 일어나지만 간혹 3-4회에서 더이상 갱신 안 될때도 있다. [Apple Document 참고][8] 
    <pre>실제 리뉴얼 주기      Sandbox 리뉴얼 주기
 1 week               3 minutes 
 1 month              5 minutes
 2 months             10 minutes 
 3 months             15 minutes 
 6 months             30 minutes 
 1 year               1 hour
</pre>

  * Apple ID : 앱에서 결제/restore 테스트를 하기에 앞서 Apple Store에서 로그아웃을 해 줘야 테스트 계정이 꼬이지 않는다.</p> 
  * 한번 결제/restore 를 한 후 에도 이상한 페이지에서 계속 login prompt가 뜬다면 finishTransaction: 함수를 제대로 불러주지 않은 것이니 코드를 체크해 본다. 
  * 한번 정기결제가 되었던 샌드박스 유저에 대해서는 해당 정기결제를 취소할 방법이 없기때문에(샌드박스 유저는 iOS설정페이지의 구독관리 페이지에 접근 불가능)다시 상품을 구매해도 오토리뉴얼이 일어나지 않는다. 이 경우에도 매번 신규 샌드박스 아이디를 만들어서 사용하는 방법 밖에 없다. [참고][9]
  * 결제 횟수나 리스토어 횟수가 너무 많은 경우 테스트가 잘 되지 않으니 새로운 sand box계정을 생성해서 테스트 한다. 
  * 가끔 SandBox 결제서버가 죽어서 아무 이유없이 결제가 안될때가 있다. 이럴때는 기다렸다가 그냥 시간이 지난 후 다시 해보는 것이 좋다.
  * TestFlight를 통해서도 Sandbox테스트가 가능하다. TestFlight에 초대를 받은 계정은 실제 Apple계정이지만 이 계정으로 결제 테스트 진행시 SandBox처럼 테스트 결제가 된다. 
      * TestFlight를 통해 설치한 베타앱에서 테스트플라이트 계정으로 IAP구매를 할경우 실제 결제창과 동일하게 나타남(Sandbox 표시 없음) 하지만 결제를 진행해도 실결제는 일어나지 않는다.

## 정기 구독 상품 가격 관련

  * 가격 변경시 주의사항 
      * 일반적인 IAP 상품의 경우 가격을 변경시 거의 즉시 반영이 가능하지만 정기 구독 상품의 경우 가격을 변경할때 해당 가격의 당일에 바로 변경하지 못하고, **다음날** 이후 부터 선택해서 예약이 가능하다. 변경 예약된 날짜 0시 0분부터 해당 가격이 반영된다. (애플 유저 계정의 timezone을 따라가는 것 같긴한데 정확히 확인은 하지 못함) 따라서 가격 변경이 예정되어있다면, 하루전에 미리 예약을 걸어두는 것이 좋다.
      * 가격을 인상하는 경우 기존 결제하고있던 유저들이 가격 인상에 동의하는지를 확인하는 절차를 거쳐야 다음 결제가 일어난다. (가격 인하의 경우는 유저의 액션 없이 인하된 가격으로 자동으로 결제가 된다)
  * USD가 아닌 경우 환율변동에 의해 각 티어(tier)별로 현지 가격이 달라진다. 하지만 정기 구독 상품의 경우 첫 결제가 일어난 시점의 해당티어 가격으로 계속해서 매달 결제가 일어난다. (환율 변동으로 인해 해당 티어의 가격이 오르거나 내리는 경우 다른 IAP 상품들의 경우 변경된 가격으로 결제됨)

## 인앱 정기 결제 FAQ 정리

  * 앱 내부에서 정기결제 상태를 확인하거나, 정기결제를 On/Off 할 수 있는지?
    
      * 불가능하다. 다음 링크를 통해서 유저를 아이튠스 구독 페이지로 보내서 켜고끄는 방법밖에 없다.
      * https://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/manageSubscriptions
  * 오토리뉴얼 서브스크립션의 경우 매 결제마다 Transaction ID가 새로 생성되는지? 
      * 하나의 애플 유저에당 하나의 receipt 정보가 존재하고 receipt안에 결제 건수만큼의 Transaction 정보가 존재한다.
      * 매번 결제가 일어날때마다 receipt안에 transaction ID가 새로 추가된다.
      * 서버쪽에서 마지막에 받았던 receipt으로 verify조회시 마지막 transaction 정보 + latest transaction 정보가 같이 리턴되서 돌아온다. 두가지가 다를경우 정상적으로 리뉴얼이 된것으로 보면 된다.
  * 유저가 아이튠스화면에서 정기결제를 꺼놓은 경우 
      * 서버나 앱에서 정기결제를 끌때 notification callback 있는지? 
          * 예전에는 없었어서 오직 receipt의 expire date로 확인해야했음
          * 하지만 2017년 7월18일 이후부터 receipt를 조회시 해당 유저가 정기결제 구독을 취소했는지를 알 수 있는 [field가 추가][10]되었다.
      * 만료기일 경과한 이후에 다시 정기결제를 켤 경우? 
          * 미결제 공백 기간(lapse)은 이용기간으로 치지않고, 정기결제를 켜면서 결제를 다시 시작한시점부터 서브스크립션 시작것으로 간주된다.
  * 결제 취소 여부 판단하는 방법은? 
      * receipt내의 Cancellation Date를 보고 취소여부 판단한다. Cancellation 필드가 있으면 만료일자와 관계없이 취소된 subscription이다.

## 애플 결제 서버 정기구독 관련 2017년 7월 18일 업데이트 내용<sup id="fnref-560-apple_new"><a href="#fn-560-apple_new" class="jetpack-footnote">3</a></sup>

애플에서 개발사들의 의견을 반영하여 2017년 7월 18일에 아래 기능들이 추가했다고 발표하였다. 좀더 자세한 내용은 [WWDC 2017 Advanced StoreKit][11] 세션을 참고하면 된다.

  * <a name="server_noti">Server Notification</a> 
      * iTunesConnect에 애플 결제서버로 부터 Notification을 받을 서비스 서버의 URL을 등록할 수 있다.
      * 아래 명시된 상황이 발생하면 등록된 URL로 http request가 전달되며 정기구독과 관련된 정보들을 받을 수 있다 <sup id="fnref-560-apple_noti"><a href="#fn-560-apple_noti" class="jetpack-footnote">4</a></sup>
      * Notification Type 
          * INITIAL_BUY: 첫 결제시에 발송
          * CANCEL: **애플 고객센터를 통해서 취소가된 정기 구독의 경우**에만 발송(주의: 구독 기간 중에 사용자가 구독관리 화면을 통해 구독을 취소한 경우에는 notification이 발송되지 않는다!)
          * RENEWAL: **기간이 한번 만료되었던 정기 구독건**이 다시 리뉴얼이 발생한 경우에만 발송 (정상적인 리뉴얼에 대해서는 notification이 발송되지 않는다!)
          * INTERACTIVE_RENEWAL: lapsed된 상태에서 사용자가 직접 명시적으로 다시 결제를 통해 리뉴얼을 진행한 경우에 발생
          * DID\_CHANGE\_RENEWAL_PREF: 구독 그룹내에 여러개의 IAP 상품(plan)이 존재할때 이것을 변경한 경우에 발송 (구독 그룹내 상품 변경은 다음 결제때부터 적용된다)
      * 일반적으로 server to server notification을 가장 원하는 시점이 사용자가 구독관리 화면을 통해서 구독을 취소한 케이스인데, 이 부분에 대한 notification을 보내주지 않아서 활용도가 매우 떨어지는 기능이 되어버렸다.
  * <a name="enhanced_receipt">Enhanced Receipts</a> 
      * 다음 결제일에 오토리뉴얼이 일어날 예정인지 확인 가능
      * 구독 가격을 인상한 경우 유저가 가격 조정에 동의 했는지 상태 
      * 리뉴얼 결제가 실패한경우 앱스토어가 해당 결제를 다시 재시도 하고있는지 여부
      * 구독 만료 이유(Expiration Intent) 
          * 구독 가격 인상시 조정 동의하지 않음
          * 신용카드 유효 기간 만료로 인한 결제 실패 등
      * 취소 이유 (Cancellation Reason) 
          * 애플 고객센터에 의한 취소 등

## 참고자료

[IAP auto renewal subscription 종합][12]

[IAP 정기 결제 단점 정리][13]

[Server-side Auto Renewable Subscription Receipt Verification][14]

<li id="fn-560-apple_subs">
  <a href="https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/Subscriptions.html">Apple Developer &#8211; Subscription</a>&#160;<a href="#fnref-560-apple_subs">&#8617;</a>
</li>
<li id="fn-560-apple_receipt">
  <a href="https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html">Apple Developer &#8211; Receipt Validation</a>&#160;<a href="#fnref-560-apple_receipt">&#8617;</a>
</li>
<li id="fn-560-apple_new">
  <a href="https://itunespartner.apple.com/en/apps/news/45333106?sc_cid=ITC-AP-ENREC">Apple Announcement &#8211; Server notifications and enhanced receipts for subscriptions</a>&#160;<a href="#fnref-560-apple_new">&#8617;</a>
</li>
<li id="fn-560-apple_noti">
  <a href="https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/Subscriptions.html#//apple_ref/doc/uid/TP40008267-CH7-SW13">Status Update Notifications</a>&#160;<a href="#fnref-560-apple_noti">&#8617;</a> </fn></footnotes>

 [1]: https://developer.apple.com/app-store/subscriptions/whats-new/
 [2]: #server_noti
 [3]: /uploads/2016/06/apple_app_receipt.png
 [4]: /uploads/2016/06/iap-types.png
 [5]: https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/Products.html
 [6]: http://stackoverflow.com/questions/17609347/restore-transactions-for-non-renewing-subscriptions-without-registration
 [7]: https://developer.apple.com/library/ios/documentation/StoreKit/Reference/SKPaymentTransaction_Class
 [8]: https://developer.apple.com/library/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnectInAppPurchase_Guide/Chapters/TestingInAppPurchases.html
 [9]: http://stackoverflow.com/questions/10289011/ios-sandbox-test-user-account-subscription-management
 [10]: #enhanced_receipt
 [11]: https://developer.apple.com/videos/play/wwdc2017/305/
 [12]: http://stackoverflow.com/questions/10689931/whats-the-logic-behind-auto-renewing-subscription-in-ios
 [13]: http://www.marco.org/2013/12/02/auto-renewable-subscriptions
 [14]: https://hetzel.net/2011-04-01/server-side-auto-renewable-subscription-receipt-verification/