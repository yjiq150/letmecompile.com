---
title: iOS 앱 리버스 엔지니어링 해보기
date: 2025-09-03T20:40:23+09:00
url: /ios-app-reverse-engineering-with-frida/
categories:
  - iOS
  - 프로그래밍 기타
tags:
  - iOS
  - frida
  - reverse engineering
---

관심있게 지켜보던 iOS 앱의 블루투스 관련 코드가 어떤식으로 구현이 된건지 궁금해서 직접 비슷하게 만들어보려다가 잘 안되길래 최후의 수단인 리버스 엔지니어링을 시도해 보았다.
LLM과 대화를 나눠보니 탈옥된 폰에서 `frida`를 사용하면 런타임에 코드를 후킹해서 조작하거나 로그를 찍어 분석이 가능하다고 해서 해당 방식으로 진행했던 부분 중 핵심적인 부분들만 단계별로 정리해 보았다.

## iPhone 탈옥

- iPhone 기종/CPU 칩셋, iOS 버전에따라 다른 형태의 탈옥 툴이 존재
- 테스트용 아이폰7 + iOS 15.6 기기를 dopamine 이용해서 탈옥 진행 
- 가이드 문서: https://ios.cfw.guide/installing-dopamine/
- Sileo 설치

## 탈옥 후 설정할 것

- 아이폰의 Sileo 앱을 통해서 설치할 것
    - ElleKit (Sources 탭에서 설치)
    - PreferenceLoader
    - Choicy
        - 설치 후 iOS 설정 앱에 들어가서 Choicy 선택 > Applications > frida attach 할 앱 선택 > Disable Tweak Injection
        - 이 작업을 하지 않으면 `frida -U -f com.targetapp.id` 실행시 (실행과 동시에 frida를 attach하는 방식) 바로 crash 발생하니 주의 (탈옥 후 모든 앱 실행시 inject되는 systemhook.dylib 와 충돌 발생 때문)
        - 이 작업을 하지 않아도 `frida-trace -U ProcessName` 형태로 기존 실행중인 앱에 attach 하는 것은 문제 없음
    - debugserver
        - Mac에서 lldb를 실행 후 아이폰 내의 프로스세스에 리모트로 연결해서 디버깅 하기 위해 필요
    - Frida
        - Sources 탭 > '+' 버튼 눌러서 https://build.frida.re/ 추가
        - 그 후 frida 설치
        - frida 버전을 선택해서 설치하는 것이 불가능. 여기서는 17.x버전을 사용
    - openssh
        - Mac에서 아이폰으로 SSH 접속하는데 필요
- 탈옥과정에서 계정 비밀번호 입력하는데 이 비번을 나중에 아이폰에 `mobile` 계정으로 SSH 접속할때 사용
    - 예) ssh mobile@192.168.1.x
    - root 접근이 필요한 경우라면 mobile 계정으로 접속 후 `sudo su` 로 전환

- Mac에 설치할 것
    - frida tools
        - https://frida.re/docs/installation/


## frida로 runtime analysis 진행하기

### 프로세스 이름 찾기

타겟앱을 아이폰에서 평범하게 실행 후 아래 명령어 실행

```
frida-ps -iaU

 PID  Name         Identifier                                              
----  -----------  --------------------------
6514  ExampleApp   com.example.app 
```

나중에 디버거 붙이기 위해 프로세스 이름을 알아야하는데 위에서 찾아진 PID로 다음 명령어 실행

```
frida-ps -U | grep 6514

6514 ExmapleProcess
```

여기서 찾아진 `ExmapleProcess`가 프로세스의 이름


### 앱을 새로 실행하면서 frida / lldb 붙이기

- 터미널1
    - `frida -U -f com.targetapp.id -l your-script-to-load.js` 실행
    - 너무 오래 pause되면 iOS가 해당 앱을 kill하기 때문에 다음 스텝을 빠르게 진행할 것
- 터미널2 
    - 아이폰으로 ssh 접속 후 해당 shell에서 `debugserver 0.0.0.0:1234 -a TargetProcessName` 실행
- 터미널1
    - frida 콘솔에서 `%resume` 실행
- 터미널3
    - lldb 실행
    - 리모트 디버그 서버로 연결: `process connect connect://192.168.1.33:1234` (IP 주소는 아이폰의 IP주소를 찾아서 넣기)
    - 성공적으로 연결되면 'Process 7379 stopped' 와 같은 메시지 출력됨
    - `c` 를 입력하면 stop된 실행을 다시 진행할 수 있음

- 참고 영상: [Bypassing iOS Anti Reversing Defences Using Frida](https://www.youtube.com/watch?v=VImxWrKB3Vo)

### 이미 실행중인 앱에 frida 붙이기

```
frida-trace -U TargetProcessName -m "Method Pattern1" -m "Method Pattern2"
```

Obj-C 메서드 찾기위한 패턴 예시: `*[* peripheralManager*]"`, `"*[* peripheral:*]`

이렇게 찾아진 메서드들은 Web UI에서 바로 js로 hook 코드를 작성할 수 있음.
해당 작성된 코드들은 로컬 머신의 `__handlers__`에 저장되고, 나중에 재활용 가능.

만약 iOS Private API 를 후킹해보고 싶다면 아래 Private API의 헤더파일들이 완전하진 않지만 메서드 이름들을 어느정도 파악할 수 있어서 도움이 되니 참고

- [iOS Runtime Headers](https://github.com/nst/iOS-Runtime-Headers/tree/master)


### custom js코드 작성시 주의점

- 위 참고 영상에서는 frida 16 기준이라 그대로 따라하면 js오류 발생 (frida 17에서 js 코드쪽에 breaking change 존재)
    - 예)
        - 16버전: Module.getExportByName(null, "NSLog")
        - 17버전: Process.mainModule.getExportByName("NSLog")
- LLM에 인터셉트 코드 작성을 요청 할 때도 항상 17버전 기준으로 작성해달라고 요청해야함. LLM이 최신버전 문서를 잘 모르는 느낌이면 Context7 MCP를 이용.


## 실제 분석 예시

아래 예시들은 '이미 실행중인 앱에 frida 붙이기' 방식은 아니고 '앱을 새로 실행하면서 frida 붙이기'방식을 사용할때 기준의 코드임. 

## 예시 1: 클래스의 모든 메서드 순회하기

`$ownMethods` 를 사용해서 특정 클래스의 메서드를 모두 순회 가능
특정 함수의 argument가 어떤 값으로 호출되는지를 확인할 수 있음.

```javascript
ObjC.classes.CBPeripheralManager.$ownMethods.forEach(function(method) {
      Interceptor.attach(ObjC.classes.CBPeripheralManager[method].implementation, {
          onEnter: function(args) {
              console.log('[CBPeripheralManager]', method, 'onEnter');
              if (method.toString() == '- setDelegate:') {
                var delegateObj = new ObjC.Object(args[2]); // args[0] = self, args[1] = _cmd, args[2] = delegate
                console.log("[*] CBPeripheralManager init delegate:", delegateObj.toString(), "class:", delegateObj.$className);

                // Hook the delegate’s methods
                delegateObj.$ownMethods.forEach(function(m) {
                    try {
                        Interceptor.attach(delegateObj[m].implementation, {
                            onEnter: function() {
                                console.log("<Delegate> called:", m);
                            }
                        });
                    } catch (e) {
                        console.log("[!] Could not hook", m, e);
                    }
                });
              }
          },

          onLeave: function(retval) {
            console.log('[CBPeripheralManager]', method, 'onLeave');
          }

      });
  });
```

## 예시 2: 콜스택 확인하기 


```javascript
 // Safe hooking with existence checks
  var CBPeripheralManager = ObjC.classes.CBPeripheralManager;

  // Hook the public addService method
  if (CBPeripheralManager['- addService:']) {
      Interceptor.attach(CBPeripheralManager['- addService:'].implementation, {
          onEnter: function(args) {
              var service = ObjC.Object(args[2]);
              console.log('\n[addService] Called');
              console.log('  UUID:', service.UUID().UUIDString());
              console.log('  Primary:', service.isPrimary());

              // Get call stack to see where it's called from
              var backtrace = Thread.backtrace(this.context, Backtracer.ACCURATE);
              console.log('  Call stack:');
              backtrace.slice(0, 10).forEach(function(addr) {
                  var symbol = DebugSymbol.fromAddress(addr);
                  if (symbol.name) {
                      console.log('    -', symbol.name);
                  }
              });
          },
          onLeave: function(retval) {
              console.log('[addService] Completed');
          }
      });
  }
```

출력결과:

```
 [addService] Called
    UUID: 00001812-0000-1000-8000-00805F9B34FB
    Primary: true
    Call stack:
      - 0x19d24 (0x100019d24)
      - 0x21cec (0x100021cec)
      - 0x157cc (0x1000157cc)
      - -[CBPeripheralManager handleServiceAdded:]
      - -[CBPeripheralManager handleMsg:args:]
      - -[CBManager xpcConnectionDidReceiveMsg:args:]
      - __30-[CBXpcConnection _handleMsg:]_block_invoke
      - _dispatch_call_block_and_release
      - _dispatch_client_callout
      - _dispatch_lane_serial_drain$VARIANT$mp
  [addService] Completed
```

## 예시 3: 전달된 argument 들을 타입에 맞게 변환하여 출력하기  

`CBXpcConnection` 와 같이 특정 클래스의 함수를 후킹 하려고할 때 해당 argument들의 타입을 보고싶다면 아래 처럼 사용.

```javascript
console.log(CBXpcConnection['- sendMsg:args:'].method)
```

런타임에 타입정보가 없다보니 argument에 전달된 값을 실제 타입에 맞게 변환해서 출력이 어려운 경우가 종종 발생. 아래 코드로 어느정도 대응 가능 

```javascript
var CBXpcConnection = ObjC.classes.CBXpcConnection;
Interceptor.attach(CBXpcConnection['- sendMsg:args:'].implementation, {
    onEnter: function (args) {
        console.log('[CBXpcConnection sendMsg:args:]');
        
        // Safely check arg[2] (first parameter - msg)
        console.log('arg[2] pointer:', args[2]);
        
        if (args[2].isNull()) {
            console.log('arg[2] is NULL');
        } else {
            // Try to read it as an ObjC object safely
            try {
                // First check if it looks like an object
                var isa = args[2].readPointer();
                console.log('arg[2] isa pointer:', isa);
                
                // Only try to create ObjC.Object if it seems valid
                if (!isa.isNull()) {
                    var obj = new ObjC.Object(args[2]);
                    console.log('arg[2] as object:', obj);
                    console.log('arg[2] class:', obj.$className);
                }
            } catch (e) {
                console.log('arg[2] is not an ObjC object, trying other interpretations...');
                
                // Maybe it's a C string?
                try {
                    var str = args[2].readUtf8String();
                    if (str) {
                        console.log('arg[2] as C string:', str);
                    }
                } catch (e2) {}
                
                // Maybe it's a number?
                try {
                    console.log('arg[2] as int:', args[2].toInt32());
                } catch (e3) {}
                
            }
        }
        
        // Safely check arg[3] (second parameter - args)
        console.log('\narg[3] pointer:', args[3]);
        
        if (args[3].isNull()) {
            console.log('arg[3] is NULL');
        } else {
            try {
                var isa = args[3].readPointer();
                console.log('arg[3] isa pointer:', isa);
                
                if (!isa.isNull()) {
                    var obj = new ObjC.Object(args[3]);
                    console.log('arg[3] as object:', obj);
                    console.log('arg[3] class:', obj.$className);
                    
                    // If it's an array, safely iterate
                    if (obj.respondsToSelector_('count')) {
                        var count = obj.count();
                        console.log('Array count:', count);
                        for (var i = 0; i < Math.min(count, 5); i++) { // Limit to first 5 items
                            try {
                                console.log('  Item', i + ':', obj.objectAtIndex_(i));
                            } catch (e) {
                                console.log('  Item', i + ': [failed to read]');
                            }
                        }
                    }
                }
            } catch (e) {
                console.log('arg[3] is not an ObjC object');
                
                // Try other interpretations
                try {
                    console.log('arg[3] as int:', args[3].toInt32());
                } catch (e2) {}
              
            }
        }
    }
});
```
