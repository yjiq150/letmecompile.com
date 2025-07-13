---
title: 모바일 디바이스간 P2P 연결 및 데이터 전송 방법
date: 2018-12-15T04:05:47+00:00
url: /p2p-connection-for-mobile-device/
dsq_thread_id:
  - 7106512360
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/how-cloudflare-works/"     class="post-739"><span class="crp_title">클라우드플레어(Cloudflare) 동작 원리</span></a></li><li><a href="https://www.letmecompile.com/kafka-consumer-offset-reset/"     class="post-786"><span class="crp_title">카프카(Kafka) Consumer offset reset 방법</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - iOS
  - 프로그래밍 기타

---
iOS/Android 모바일 디바이스들 사이에서 P2P 형태로 데이터 전송을 하는 방법에 대해 리서치 할 일이 있어서 알아본 내용을 간단히 공유합니다.

## 네트웍 타입별 Android/iOS P2P 연결 가능성 체크

  * Bluetooth 
      * 애플 MFi인증 디바이스만 iPhone/iPod과 Bluetooth 연결 가능 
      * MFi는 apple에서 accessary 제조업체들이 만든 하드웨어를 인증하는 제도 (안드로이드는 인증되지 않음)
  * Bluetooth Low Energy 
      * BLE는 애플 MFi인증 제약이 존재하지 않아서 안드로이드/iOS간 연결 가능
      * BLE는 말그대로 저전력으로 설계된 spec이기때문에 전송 속도가 매우 느림
      * iOS는 모두 지원하지만 안드로이드 디바이스별로 지원 스펙이 다름 (제대로 지원안하는 경우도 있음)
      * BLE의 최대 전송속도 
          * Apple guide line을 따르는 BLE 디바이스라면 디바이스에따라 이론적인 최대 전송속도 8~32kbps 정도가 한계 
              * packet interval, MTU size 파라메터에 맞춰 application이 전송하는 데이터를 완벽하게 fit 시켰을때 나올 수 있는 이론적인 수치. 실제로는 더 느릴것 같음
          * <https://devzone.nordicsemi.com/question/3440/how-do-i-calculate-throughput-for-a-ble-link/>
  * WiFi 
      * 동일 네트워크 라우터에 접속한 디바이스들사이에서 P2P 연결 가능 
          * multicast를 지원하는 라우터인경우 mDNS(ex: Apple Bonjour)를 이용하여 자동으로 동일 네트웍상의 디바이스를 찾아 주는 것이 가능
          * 개인적으로 사용하는 office/home wifi가 아닌 WiFi hotspot 같은 공용 라우터의 경우 P2P 연결이 안될 가능성이 높음.
      * 뮤직 서버에서 P2P연결 시작만 중계하는 
          * NAT 뒤에 연결된 디바이스들도 연결이 가능하긴 함 UDP 홀펀칭 등의 방식을 사용하면 어느정도 커버가능하긴하지만 100% 연결 보장은 힘듦
  * 참고자료 
      * [Android/iOS Local Radio Interop][1]
      * 

## 3rd party frameworks

  * [Google Nearby][2] 
      * Bluetooth, BLE, WiFi 등의 연결방식을 상황에따라 적절히 조합해서 근처에 있는 device를 찾아서 연결해주는 framework
      * 아래 두가지 API set이 있는데 상황/용도에따라 다르게 사용
      * Message API 
          * iOS/Android
          * small payload only through google server (devices do have to be connected to the Internet).
          * 음원전송은 불가능
      * Connection API 
          * Android ONLY
          * P2P connection: message, file, stream transfer support
          * 음원전송에 적합
  * [http://p2pkit.io][3] 
      * 유료 framework 
          * $0.01/ MAU
      * Android/iOS 둘다 서포트 하지만 GoogleNearBy 와 동일하게 대역폭한계로 음원전송에는 부적합해보임 
          * 20 messages per second and cannot exceed the size of 100KB

 [1]: http://www.goland.org/thaliiosandroidinterop/
 [2]: https://developers.google.com/nearby/
 [3]: http://p2pkit.io/