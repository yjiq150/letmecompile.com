---
title: Bluetooth Connection을 사용하는 멀티플랫폼 앱 개발기
date: 2015-03-15T03:03:21+00:00
url: /bluetooth-connection을-사용하는-멀티플랫폼-앱-개발기/
dsq_thread_id:
  - 3596134474
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/pake-srp-protocol/"     class="post-802"><span class="crp_title">PAKE와 SRP Protocol을 이용한 인증</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li><li><a href="https://www.letmecompile.com/shotcut-linux-server-video-generation/"     class="post-753"><span class="crp_title">Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - Cocoa
  - Thrift
tags:
  - 블루투스 통신
  - bluetooth communication
  - thrift with bluetooth

---
블루투스 연결을 사용하는 멀티플랫폼(Mac, Window, Android, Windows Store app) 앱 개발을 하면서 겪은 경험과 노하우들을 정리해 보았다. 아직 완전히 개발이 끝나지 않았기때문에 포스트 내용에도 부족한 점들이 많이 있지만, 일단은 먼저 경험을 공유하는 것이 중요한 것같아서 포스팅을 해둔다.

# Bluetooth 연결 기본

## 블루투스는 디바이스를 어떻게 찾는가?

블루투스 디바이스는 자신이 제공할 수있는 서비스를 무선 네트워크상에 publish해서 다른 디바이스들이 검색할 수 있도록 하는데, 이때 사용되는 것이 Service Discovery Protocol(SDP) 이다.

예를들어 블루투스 키보드는 자신이 HID 서비스(입력장치)가 가능하다고 무선네트워크상에 계속 신호를 보내고있는 상태인 것이고, 컴퓨터에서 주변 블루투스 장치를 검색할때 이 신호를 인식하여 &#8220;블루투스 키보드 장치가 있다&#8221;는 사실을 알게되는것이다. 이때 이러한 서비스들을 유니크하게 구분하기위해 UUID를 사용하게된다. UUID의 기존에 이미 정의된 것들을 사용할 수 있으며, 새롭게 임의로 UUID를 생성해서 사용하는것도 가능하다.

## TCP 소켓처럼 블루투스를 사용가능한가?

시리얼 통신방식을 블루투스에서 에뮬레이트하는 RFCOMM을 사용할 경우 거의 TCP 소켓을 사용하는것처럼 비슷하게 사용할 수 있다.  
실제 Android의 경우 `BluetoothSocket` 이라는 클래스를 제공하며 TCP와 거의 동일한 인터페이스를 가지기때문에  
기존 TCP 통신 사용하던 어플리케이션의 스트럭쳐를 블루투스에서 그대로 사용하는것이 가능하다.

참고: RFCOMM의 UUID는 다음과 같다.

> RFCOMM&#8217;s UUID 00001101-0000-1000-8000-00805F9B34FB 

그렇다면 이제 실제로 [Thrift][1]를 이용하여 각 플랫폼별로 어떻게 블루투스 통신을 하는지 살펴보도록 하자.

# 블루투스 통신에서 Thrift 사용하기

본 글에서는 Android 디바이스(클라이언트)에서 Windows PC 또는 Mac OS(서버)로 블루투스를 이용하여 데이터를 주고받는 시나리오에 대해서 설명해보려고 한다. 필자는 원래 WiFi를 통한 TCP 연결(Thrift 기반의 RPC)을 통해서 데이터를 주고받고 있었지만, 동일한 동작을 Bluetooth를 통해서도 가능하게 하기 위해서 이 리서치를 시작하게 되었다.

## Use thrift over Bluetooth connection

기본적으로 thrift에서 제공하는 Objective-C 라이브러리에는 TCP 소켓 기반의 전송계층(transport layer) 밖에 존재하지 않는다. 따라서 두 디바이스간의 컨넥션을 Ethernet이 아닌 블루투스를 통해서 열었을 경우 TCP기반의 통신이 아니기 때문에 따로 블루투스에 맞게 전송계층을 구현해주어야 한다. 본 글에서는 시리얼 프로토콜 에뮬레이션을 위한 블루투스의 프로파일 중 하나인 SPP(Serial Port Profile)을 이용하여 전송계층을 구현하는데 (SPP는 블루투스의 RFComm 레이어를 이용하여 통신을 하기때문에 거의 비슷한 의미로 생각하면 된다), 이를 위해 서버/클라이언트에 각각 어떤방식의 구현이 필요한지 살펴보도록 하자.

## Client side (Android)

Android에서 블루투스를 이용한 데이터 통신을 하기 위해서는 블루투스 스택중 RFComm 레이어를 이용하는 `BluetoothSocket`을 이용해야 한다. (현재 안드로이드 SDK 중 자바 레벨의 인터페이스에서 L2CAP은 사용이 불가능한것으로 보인다) 이 블루투스 소켓으로부터 Input/Output 스트림을 얻어와서 일반적인 TCP소켓기반의 통신과 거의 동일한 인터페이스로 손쉽게 이용이 가능하다. 해당 샘플코드는 안드로이드 예제중 하나인 BluetoothChat을 검색하여 살펴보면된다.

기존 TCP소켓과 인터페이스 자체가 동일하기 때문에 블루투스 소켓을 생성한 이후에는 아무런 수정없이 Thrift 자바(java) 라이브러리에 포함된 `TIOStreamTransport`를 이용하여 Thrift 클라이언트를 손쉽게 생성이 가능하다.

## Server side (Mac OS)

### Cocoa IOBluetooth.framework 라이브러리의 특징<sup id="fnref-474-apple"><a href="#fn-474-apple" rel="footnote">1</a></sup>

Mac OS에서 블루투스로 데이터 통신을 하기위해서는 `IOBluetooth` 프레임웍에 포함된 `BluetoothRFCOMMChannel` 클래스를 이용하면된다. `BluetoothRFCOMMChannel` 말고도 한단계 아래 레이어인 `BluetoothL2CAPChannel` 또한 지원하지만, 안드로이드 클라이언트쪽에서 L2CAP 사용이 불가능하다보니 선택의 여지가 없이 RFCOMM을 이용할 수밖에 없다.

`BluetoothRFCOMMChannel`의 경우 동기/비동기 쓰기(`writeSync:`, `writeAsync:`) 동기 읽기(synchronous read) 오퍼레이션을 지원해주지 않는다. 대신 runloop기반으로 채널상에 읽을수 있는 data가 준비되면 delegation을 호출하여 데이터를 읽어들이는 방식으로 동작한다.

Thrift 서버의경우 기본적으로 해당 RPC 커넥션에대해 하나의 스레드를 생성한 후, 해당 스레드에서 동기 읽기/쓰기(synchronous read/write)를 사용하여 모든 작업을 수행하기때문에 `IOBluetoothChannel` 채널을 트랜스포트 레이어로 바로 사용하기 쉽지 않다. 따라서 블루투스 채널을 Thrift의 트랜스포트 레이어로 사용하기위한 려면 몇가지 트릭들이 필요하다.

### Thrift를 위한 BluetoothTransport 레이어 구현하기

블루투스 채널에 write operation을 하기위해 Thrift에서 제공하는 라이브러리 중에 `NSStreamInput/Output`을 인자로 받아들이는 `TNSStreamTransport` 를 조금 변형하여 inputStream은 그대로 두고, outputStream을 `IOBluetoothRFCOMMChannel` 로 변경하여 다음과 같이 구현하였다.

#### TIOBluetoothTranport.h

    #import <Foundation/Foundation.h>
    #import "TTransport.h"
    
    
    @class IOBluetoothRFCOMMChannel;
    
    @interface TIOBluetoothTransport : NSObject <TTransport> {
        NSInputStream * mInput;
    
        IOBluetoothRFCOMMChannel * _channel;
    }
    
    - (id) initWithInputStream: (NSInputStream *) input
                       channel:(IOBluetoothRFCOMMChannel *) channel;
    
    @end
    

#### TIOBluetoothTranport.m

    #import "TIOBluetoothTransport.h"
    #import "TTransportException.h"
    #import "TObjective-C.h"
    #import <IOBluetooth/IOBluetooth.h>
    
    @implementation TIOBluetoothTransport
    
    - (id) initWithInputStream:(NSInputStream *) input
                       channel:(IOBluetoothRFCOMMChannel *) channel;
    {
      self = [super init];
      mInput = input;
      _channel = channel;
      return self;
    }
    
    
    - (int) readAll: (uint8_t *) buf offset: (int) off length: (int) len
    {
      int got = 0;
      int ret = 0;
      while (got < len) {
        ret = [mInput read: buf+off+got maxLength: len-got];
        if (ret <= 0) {
          @throw [TTransportException exceptionWithReason: @"Cannot read. Remote side has closed."];
        }
        got += ret;
      }
      return got;
    }
    
    - (void) write: (const uint8_t *) data offset: (unsigned int) offset length: (unsigned int) length
    {
        IOReturn result = 0;
    
        result = [_channel writeSync:(void*)data length:length];
    
        if (result != kIOReturnSuccess) {
            NSError* error = [[NSError alloc] initWithDomain:@"SSBluetoothWriteError" code:result userInfo:nil];
          @throw [TTransportException exceptionWithReason: @"Error writing to transport output stream."
                                                    error: error];
        }
    }
    
    - (void) flush
    {
      // no flush for you!
    }
    
    @end
    

이제 쓰기동작은 잘 정의 되었으니 `IOBluetoothRFCOMMChannel` 에서 delegate 형태로 수행되는 read 오퍼레이션을 `TIOBluetoothTransport`의 입력스트림(input stream)으로 어떻게 리다이렉션(redirection) 할까?

채널의 `rfcommChannelData:data:length:` delegate가 호출되는 순간 채널로부터 읽어진 데이터를 출력 스트림에 쓰고(write), 다른쪽에서는 출력스트림에서 써놓은 데이터를 입력 스트림으로 생각하여 읽기(read) 오퍼레이션을 수행하도록 하면, 이 입력스트림을 위의 `TIOBluetoothTransport`에 inputStream으로 넣어줄 수있다.

방금 위에서 설명한 동작을 실현시켜주는 `CFStream` 라이브러리 함수가 다행히도 존재한다. `CFStreamCreateBoundPair()`를 이용하면 `NSOutputStream`과 `NSInputStream`을 중간 버퍼를 두고 연결하는것이 가능해진다. 이 함수를 사용해서 두 스트림을 연결하면 output stream에 write 할때 내부적으로 중간 버퍼에 저장이 되어있다가 페어링된 inputStream에서 read를 할때 해당 버퍼에서 읽어들이게된다. 자세한 사용법은 애플에서 제공하는 예제코드인 \[SimpleURLConnections\] (http://developer.apple.com/library/ios/#samplecode/SimpleURLConnections/Introduction/Intro.html#//apple_ref/doc/uid/DTS40009245)를 참조하면된다.

  * 팁1: 스트림 바운드 페어를 쓰지 않더라도 이 상황을 생산자-소비자(Producer-Consumer) 패턴으로 생각하여 BlockingQueue를 사용하여 해결하는 방법도 가능하다. delegate가 불리는 시점에 queue에 데이터를 add하고, 다른쪽에서는 read operation때 해당 queue에서 데이터를 꺼내가능 방식으로 구현하면된다.  
    java에서는 BlockingQueue가 기본라이브러리에 포함되어있지만 아쉽게도 Objective-C에는 없다. Objective-C에서도 이러한 BlockingQueue를 어렵지않게 구현가능<sup id="fnref-474-BQ"><a href="#fn-474-BQ" rel="footnote">2</a></sup>하니 참고하길 바란다. 좋은방법은 아니지만 비슷한 접근법으로 NSCondition의 wait & signal 을 이용하는것도 가능해 보인다<sup id="fnref-474-waitsignal"><a href="#fn-474-waitsignal" rel="footnote">3</a></sup>.</p> 
  * 팁2: `CFStreamCreateBoundPair()`를 사용할경우 버퍼의 사이즈와 버퍼에 쓰여지는 데이터의 사이즈와 횟수에따라 버퍼 플러쉬가 되지 않아 문제가 생길 수 있으니 다음 링크 참조하도록 하자. [Stream Pair buffer size problem][2]

### 데이터 전송 병목(bottleneck) 해결

아직 완전히 해결하지 못하여 관련자료만 소개한다.  
RFCOMM은 시리얼포트 통신을 에뮬레이션해야하기때문에 스루풋에 한계가 있는것으로 보인다.

[stackoverflow: android-bluetooth-serial-rfcomm-low-baud-rate-slow-transmission][3]

한정된 스루풋에서 데이터 전송을 잘하려면 결국 데이터를 최소화 하는수밖에 없는데 이를 위해 Thrift에서 패킷사이즈를 적게사용하는 `CompactProtocol` 사용하는것을 추천한다.

### 참고자료

[블루투스 크로스 플랫폼 라이브러리 bluecove][4]  
코코아 기반으로 low level bluetooth 코드를 짜기위한 좋은 샘플코드들이 많이 있음.

## Server side (Windows)

### Thrift 라이브러리 적용

Thrift 라이브러리는 C#을 지원하므로 C# 기반의 코드와 연동을 하는것이 수월한 편이다.  
하지만 Thrift C# Server에서 사용하는 Socket 라이브러리가 윈도우 PC에서만 이용가능하기때문에 Windows Store App을 만드는 것이 불가능하다. 이 에러를 피해서 사용하도록 주석처리해둔 프로젝트가 존재해서(https://thriftwinrt.codeplex.com/) 필자는 이것이 기반하여 일단 라이브러리를 연동하였다.

하지만 Windows Store App에 사용가능한 소켓 라이브러리에는 동기방식의 API가 존재하지 않다보니 Thrift 서버를 돌릴 코드가 없는상화이 되어버렸고, 이를 해결하기위해 강제로 비동기 API를 동기화해서 사용하는 방법을 적용했다.

즉, 소켓 라이브러리가 C#의 `async`, `await` 키워드를 이용하여 비동기방식으로 구현되어있는데 이를 우회하기 위해 필자는 Semaphore를 이용하여 async 호출 끝부분에 block을 시켜서 해당 호출이 실제로 끝날때까지 waiting을 하도록 라이브러리를 수정해서 사용하였다. 효율적이진 않지만 최소한의 코드수정으로 Thrift 라이브러리를 이용하기 위해 어쩔 수 없는 선택이었다. 다음 `TSocket`코드를 실제 Thrift 0.9.2 원래 소스와 `diff` 를 돌려가면서 참조하면 쉽게 이해가 갈것이다.

    using System;
    using Windows.Networking.Sockets;
    using System.IO;
    //using System.Net.Sockets;
    
    namespace Thrift.Transport
    {
        public class TSocket : TStreamTransport
        {
            //private TcpClient client = null;
            private string host = null;
            private int port = 0;
            private int timeout = 0;
            private StreamSocket client;
    
            //public TSocket(TcpClient client)
            //{
            //    this.client = client;
    
            //    if (IsOpen)
            //    {
            //        inputStream = client.GetStream();
            //        outputStream = client.GetStream();
            //    }
            //}
    
            public TSocket(string host, int port)
                : this(host, port, 0)
            {
            }
    
            public TSocket(string host, int port, int timeout)
            {
                this.host = host;
                this.port = port;
                this.timeout = timeout;
    
                InitSocket();
            }
    
            private void InitSocket()
            {
                //client = new TcpClient();
                client = new StreamSocket();
                //client.ReceiveTimeout = client.SendTimeout = timeout;
                //client.Client.NoDelay = true;
            }
    
            public int Timeout
            {
                set
                {
                    //client.ReceiveTimeout = client.SendTimeout = timeout = value;
                }
            }
    
            //public TcpClient TcpClient
            //{
            //    get
            //    {
            //        return client;
            //    }
            //}
    
            public string Host
            {
                get
                {
                    return host;
                }
            }
    
            public int Port
            {
                get
                {
                    return port;
                }
            }
    
            public override bool IsOpen
            {
                get
                {
                    if (client == null)
                    {
                        return false;
                    }
    
                    if (inputStream == null || outputStream == null)
                    {
                        return false;
                    }
    
                    return true;
    
                    //return client.Connected;
                }
            }
    
            public async override void Open()
            {
                if (IsOpen)
                {
                    throw new TTransportException(TTransportException.ExceptionType.AlreadyOpen, "Socket already connected");
                }
    
                if (String.IsNullOrEmpty(host))
                {
                    throw new TTransportException(TTransportException.ExceptionType.NotOpen, "Cannot open null host");
                }
    
                if (port <= 0)
                {
                    throw new TTransportException(TTransportException.ExceptionType.NotOpen, "Cannot open without port");
                }
    
                if (client == null)
                {
                    InitSocket();
                }
    
                try
                {
                    await client.ConnectAsync(new Windows.Networking.HostName(host), port.ToString());
    
                    inputStream = client.InputStream.AsStreamForRead();
                    outputStream = client.OutputStream.AsStreamForWrite();
    
                    Semaphore.Release();
                }
                catch (Exception exception)
                {
                    // the Close method is mapped to the C# Dispose
                    client.Dispose();
                    client = null;
    
                    inputStream = null;
                    outputStream = null;
    
                    Semaphore.Release();
                }
    
                //client.Connect(host, port);
                //inputStream = client.GetStream();
                //outputStream = client.GetStream();
            }
    
            public override void Close()
            {
                base.Close();
                if (client != null)
                {
                    //client.Close();
                    client.Dispose();
                    client = null;
                }
            }
    
            #region " IDisposable Support "
            private bool _IsDisposed;
    
            // IDisposable
            protected override void Dispose(bool disposing)
            {
                if (!_IsDisposed)
                {
                    if (disposing)
                    {
                        if (client != null)
                            ((IDisposable)client).Dispose();
                        base.Dispose(disposing);
                    }
                }
                _IsDisposed = true;
            }
            #endregion
        }
    }
    

`TTransport.cs` 에는 다음과같이 세마포어를 초기화하는 코드를 추가했다.

    public TTransport()
    {            
        this.Semaphore = new SemaphoreSlim(0, 1);
    }
    public SemaphoreSlim Semaphore
    {
        get;
        set;
    }
    

최종적으로 실제 서버에서 컨넥션을 열때 사용하는 방식은 다음과 같다.

    // connection trying
    var protocol = new TBinaryProtocol(mTransport);
    mClient = new VTService.Client(protocol);
    
    try
    {
        mTransport.Open();
    
        // timeout or open error
        if (mTransport.Semaphore.Wait(mConnectionTimeout) == false || !mTransport.IsOpen)
        {
            connectionFailed();
            return;
        }
    }
    catch (Exception ex)
    {
        postMessage(ex.Message);
        connectionFailed();
        return;
    }
    

### 블루투스 라이브러리 적용

블루투스 연결의 경우에는 [32Feet.NET][5] 을 이용하여 구현하면된다. 문서화도 잘되어있는 편이라 그리 어렵지 않게 구현 할 수 있다.

### 참고자료

[윈도우즈 버전별 블루투스 지원 스펙][6]

<li id="fn-474-apple">
  <a href="https://developer.apple.com/library/mac/documentation/DeviceDrivers/Conceptual/Bluetooth/BT_Develop_BT_Apps/BT_Develop_BT_Apps.html">Developing Bluetooth Applications, Mac Developer Library</a>&#160;<a href="#fnref-474-apple" rev="footnote">&#8617;</a>
</li>
<li id="fn-474-BQ">
  <a href="http://www.uchidacoonga.com/2014/05/blocking-queue-in-objective-c/">Blocking Queue in Objective-C</a>&#160;<a href="#fnref-474-BQ" rev="footnote">&#8617;</a>
</li>
<li id="fn-474-waitsignal">
  <a href="http://stackoverflow.com/questions/5126175/iobluetooth-synchronous-reads">http://stackoverflow.com/questions/5126175/iobluetooth-synchronous-reads</a>&#160;<a href="#fnref-474-waitsignal" rev="footnote">&#8617;</a> </fn></footnotes>

 [1]: https://thrift.apache.org/
 [2]: http://stackoverflow.com/questions/14730613/cfstreamcreateboundpair-is-writing-4kb-data-to-stream-and-stream-will-parse-the
 [3]: http://stackoverflow.com/questions/14524866/android-bluetooth-serial-rfcomm-low-baud-rate-slow-transmission
 [4]: https://bluecove.googlecode.com
 [5]: http://32feet.codeplex.com/
 [6]: http://msdn.microsoft.com/en-us/library/windows/hardware/dn133849.aspx