---
title: JavaScript 개발자에게 Kotlin coroutine 10분만에 이해시키기
author: yjiq150
type: post
date: 2020-06-19T16:58:45+00:00
url: /kotlin-coroutine-vs-javascript-async-comparison/
dsq_thread_id:
  - 8085130791
dsq_needs_sync:
  - 1
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/regex-negative-lookahead-example/"     class="post-789"><span class="crp_title">정규식 Negative Lookahead 예제</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/regex-negative-lookahead-example/"     class="post-789"><span class="crp_title">정규식 Negative Lookahead 예제</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Kotlin
tags:
  - 코틀린
  - 코루틴
  - 자바스크립트

---
Kotlin의 coroutine은 비동기 프로그래밍을 편리하게 하기 위한 매우 강력한 툴이다. 하지만 처음 접했을 때 `suspend`, `async`, `await`, `runBlocking`, `launch`, `coroutineScope`, `suspendCoroutine` 등 생소한 용어들이 많다보니 어떤 상황에 어떤 코드를 작성해야하는지 적응하는데 시간이 많이 걸린다. 하지만 비동기 프로그래밍이라는 개념 자체는 언어가 달라지더라도 일맥상통한다. 때문에 JavaScript에서 비동기 프로그래밍을 많이 해본 사람들이라면 다양한 상황에 맞춰서 JavaScript 예제와 Kotlin coroutine 예제 코드를 나란히 놓고 비교해서 설명하는 방식으로 10분만에 이해 시킬 수 있을 것이라 확신한다.

(JavaScript 비동기 프로그래밍을 익숙하게 할 수 있는 개발자가 아니라면 이 글을 읽는 것을 추천하지 않는다.)

## JavaScript vs. Kotlin 비동기 함수 호출 방법

JavaScript에서 비동기 함수는 `async` 키워드를 함수 정의 앞에 붙이고, Kotlin coroutine에서 비동기 함수는 `suspend` 키워드를 함수 정의 앞에 붙여준다.

먼저 JavaScript 개발자에게 익숙한 아래 코드를 살펴보자.

    async function getDataFromRemoteApi() {
      const response = await apiClient.get('resource')
      return response.data
    }
    

위 코드를 Kotlin coroutine으로 작성하면 아래와 같다.

    suspend fun getDataFromRemoteApi(): MyData {
      val response = apiClient.get('resource')
      return response.data
    }
    //  위  코드에서 사용된 apiClient는 아래와 같이 구성되어있다고 가정
    class ApiClient {
        suspend fun get(path: String): Response
    }
    

  * Kotlin의 경우 JavaScript의 함수 앞에 붙는 `async` 키워드 대신 `suspend`로 바꿔서 사용하면 된다.
  * JavaScript에서도 `async` 함수를 호출하는 함수는 필수적으로 `async` 키워드를 붙여줘야한다. Kotlin의 `suspend` 키워드도 마찬가지 규칙이 적용된다.
  * 함수 내부 구현을 살펴보면 `apiClient.get('resource')` 호출시에 JavaScript와는 달리 앞에 `await` 같은 키워드를 사용하지 않고 있는 것을 알 수 있다. Kotlin의 경우 suspend fun 안에서 또다른 suspend fun 을 호출하는 경우 자동으로 해당 suspend fun의 결과가 준비될 때 까지 다음 라인이 실행되지 않는다.

이제 기본적인 호출 방법은 익혔으니 실제 발생하는 상황들에 대해서 각각 어떻게 대응하면 될지 알아보도록 하자

## 상황 1: 여러개의 suspend fun을 동시에 수행하기

위에서 살펴본 기초 예제는 비동기 요청을 하나만 수행하는 간단한 코드였지만, 실제로 코드를 작성하다 보면 여러개의 비동기 요청을 동시에 수행하는 상황도 존재한다.

단순하게 생각하면 아래 코드와 같이 작성하게 될텐데, 아래 코드의 경우 비동기 요청을 여러번 수행하지만 첫번째 요청이 끝나는 것을 기다렸다가 두번째 요청을 순차적으로 수행하기 때문에 응답을 얻는데 걸리는 시간이 오래 걸릴 수 있다.

    async function getDataFromMultipleRemoteApi() {
      const response1 = await apiClient.get('resource1')
      const response2 = await apiClient.get('resource2')
      return response1.data + response2.data
    }
    

위 코드를 Kotlin coroutine으로 작성하면 아래와 같다.

    suspend fun getDataFromMultipleRemoteApi(): MyData {
      val response1 = apiClient.get('resource1')
      val response2 = apiClient.get('resource2')
      return response1.data + response2.data
    }
    

동일하게 작성되었으니 Kotlin 에서도 위의 코드는 여전히 비효율적이다. 어떻게하면 이 상황을 해결할 수 있을까?

JavaScript에 익숙한 개발자라면 `async`/`await`와 `Promise` 개념을 이용하여 다음과같은 코드를 생각해낼 것이다.

    async function getDataFromMultipleRemoteApi() {
      // async 함수 앞에 await 키워드를 빼면 async 함수의 수행이 완료될때까지 기다리지 않는다. 
      // 즉, promise를 바로 리턴한후 코드 실행을 계속 진행한다.
      const promise1 = apiClient.get('resource1') 
      const promise2 = apiClient.get('resource2')
    
      // promise들이 모두 완료되기를 기다린다.
      const [response1, response2] = await Promise.all([promise1, promise2])
      return response1.data + response2.data
    }
    

`Promise`를 이용해서 여러개의 비동기 요청을 훨씬 효율적으로 수행 할 수 있게 되었다. 예제의 비동기 요청은 겨우 2개지만, 비동기 요청 수가 많아질 경우 작업이 완료되는 속도가 훨씬 빨라질 것이다.

위 코드를 Kotlin coroutine으로 작성하면 아래와 같다.

    suspend fun getDataFromMultipleRemoteApi(): MyData {
      // async {} 를 사용하려면 새로운 coroutinScope를 생성해서 감싸줘야한다.
      val deferedList = coroutineScope  {
        // async 로 감싸면 Deferred 객체가 바로 리턴되고 코드 실행을 계속 진행한다.
        val deferred1 = async { apiClient.get('resource1') }
        val deferred2 = async { apiClient.get('resource2') }
        listOf(deferred1, deferred2)
      }
    
      // JavaScript의 Promise.all(...) 과 동일한 역할 수행
      val deferred = deferedList.awaitAll()
      val response1 = deferred[0]
      val response2 = deferred[1]
      return response1.data + response2.data
    }
    

결국 JavaScript의 `Promise`는 Kotlin coroutine의 `Deferred` class 와 개념적으로 동일한 것이고 (Java 에서는 `Future` class 사용) , 해당 개념이 지원하는 근본적인 operation 들은 동일함을 알 수 있다.

## 상황 2: callback 기반의 코드 흐름에서 suspend fun 호출하기

JavaScript에서 최신 코드나 라이브러리들은 대부분 async/await 또는 promise를 사용할 수 있지만, 해당 방식을 지원하지 않거나, event기반의 프로그래밍을 하다보면 callback 방식만 지원하는 코드를 작성해야하는 경우가 종종 있다. 이런 상황은 Kotlin에서도 callback 기반 코드 흐름 중에서 suspend fun을 호출하려고 하면 동일하게 발생한다.

Javscript의 경우 async 함수의 요청의 결과를 callback쪽으로 넘겨주기 위해서 promise를 사용하여 아래와같이 처리할 수 있다.

    // 함수 자체에는 리턴값이 없고 callback function을 호출할 때  parameter를 통해서 결과를 돌려준다
    function getDataFromRemoteApi(callback) {
      apiClient.get('resource')
          .then(response => callback(null, response.data))
    }
    

비슷한 경우 Kotlin에서는 아래와 같이 처리 할 수 있다.

    fun getDataFromRemoteApi(callback: (MyData) -> Unit) {
      // suspend 키워드가 붙지않은 함수내에서는 suspend fun인 apiClient.get을 호출할 수 없다.
      // 이 경우 suspend func을 실행할 수 있도록 코루틴을 생성하고 해당 스코프 안에서 suspend fun을 수행한다.
      launch {
          val response = apiClient.get('resource')
          callback(response.data)
      }
      // apiClient.get 수행이 완료되기 전에 이미 이 코드가 실행됨
      logger.info("When will this code executed?")
    }
    

위 예제를 보면 suspend 키워드가 붙지않은 일반적인 함수에서는 suspend fun을 호출할 수 없다는 것을 알 수 있고, `launch` 함수를 이용하여 새롭게 코루틴을 생성하고 해당 코루틴 스코프 내에서 비동기 요청을 진행하는 것을 알 수 있다.

Kotlin coroutine에는 `launch` 함수 말고도 아래처럼`runBlocking` 이라는 함수도 있는데 어떤 차이점이 있는지 아래 로그를 찍는 함수가 실행되는 시점을 보면 금방 알아 챌 수 있다.

    fun getDataFromRemoteApi(callback: (MyData) -> Unit) {
      runBlocking {
          val response = apiClient.get('resource')
          callback(response.data)
      }
      // apiClient.get 수행이 완료되고 runBlocking내부의 나머지 코드가 실행이 완전히 끝난 후에 이 코드가 실행됨
      logger.info("When will this code executed?")
    }
    

## 상황 3: suspend fun 안에서 callback style 함수의 결과를 기다려야 한다면?

JavaScript의 async 함수 내에서 callback으로 결과를 돌려주는 함수를 기다리려면 보통 다음과 같이 promise로 callback을 wrapping하면된다.

    async function getDataFromRemoteApi() {
      const promise = new Promise((resolve, reject) => {
        apiCallbackClient.get('resource', (response) => {
          resolve(response)
        })
      })
    
      const response = await promise 
      return response.data
    }
    

동일하게 Kotlin에서 suspend 함수 내부에서 callback 함수를 기다리려면 아래와 같이 `suspendCoroutine`을 이용 할 수 있다.

    suspend fun getDataFromRemoteApi(): MyData {
        // 코루틴 스코프를 잠시 멈추고 결과를 기다리는 의미의 suspendCoroutine 함수를 사용
       val response = suspendCoroutine { cont: Continuation<Response>
           // 
           logger.info("Call order: 0")
            apiCallbackClient.get('resource') { response ->
                 logger.info("Call order: 2")
                 // 성공적으로 결과를 받아온 경우 결과값과 함께 코루틴의 진행을 재개
                 cont.resumeWith(response)
            }
            logger.info("Call order: 1")
        }
        logger.info("Call order: 3")
        return response.data
    }
    
    //  위  코드에서 사용된 apiCallbackClient는 아래와 같이 구성되어있다고 가정
    class ApiCallbackClient {
        fun get(path: String, (Response) -> Unit)
    }
    

실제 위 코드가 수행 될때 로그 메시지는 Call order 0, 1, 2, 3 순서로 으로 출력 될 것이다.

## 마무리

위에서 비교해가면서 살펴본 바와 같이 특정 상황에 맞닥드렸을 때 해당 상황이 어떤 상황인지 파악 할 수 있다면 Kotlin coroutine 프로그램을 작성하다가 발생하는 대부분의 상황들을 어느정도 해쳐나갈 수 있을것이고, 코루틴을 통해서 더 나은 코드를 작성 할 수 있게 될 것이다. (다만 coroutine 내부에서 exception이 throw되는 경우 에러처리하는 방식이 좀 까다로운 편이라 이부분은 본 글에서는 다루지 않고 차 후에 별도로 정리해 볼 예정이다.)

JavaScript의 유명한 문제였던 비동기 코드 작성시 발생하는 콜백 지옥(callback hell)은 언어 문법 차원의 `async`/`await` 키워드 도입과 함께 대부분 해결되었다. Java에서 비동기 코드를 작성하려면 언어 문법차원의 지원이 없다보니 `Future` 등의 클래스를 통해서 가독성 떨어지는 복잡한 비동기 코드를 작성해야했지만, 이런 문제는 Kotlin coroutine을 도입하면 대부분 해결되리라 생각한다.

기존에도 이미 JVM기반의 비동기 웹서버들이 많이 나와있었지만 코드 생산성이 많이 떨어질 수 밖에 없어서 채택이 더딘 편이었는데 Kotlin coroutine을 통해 앞으로 JVM 기반 웹서버들도 가독성/생산성과 성능 두마리 토끼를 다 잡을 수 있지 않을까 생각된다.