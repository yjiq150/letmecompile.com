---
title: Shotcut을 이용하여 리눅스 서버에서 템플릿 기반의 동영상 만들기
author: yjiq150
type: post
date: 2018-05-22T19:08:07+00:00
url: /shotcut-linux-server-video-generation/
dsq_thread_id:
  - 6687100764
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 프로그래밍 기타
  - 웹 개발
tags:
  - shotcut
  - linux video editor
  - cli video generator

---
웹사이트 상에서 미리 제작된 영상 템플릿을 제공하고 이에 맞게 사용자가 이미지나 텍스트만 바꿀 수 있도록 툴을 제공한 후 사용자가 만들어낸 데이터 모델을 기반으로 영상을 만들어내는 video generator 기능을 구현하려면 어떻게 해야할까?

일단 웹사이트 프론트엔드에서 이미지를 업로드하고, 텍스트를 입력하고 배치하는 일들은 일반적인 웹사이트 제작에 사용되는 Vue.js나 React 등을 사용해서 빠르게 만들 수 있다. 이렇게 사용자에게 영상을 구성할 수 있는 툴을 제공하고, 해당 툴을 이용해서 생성된 모델을 백엔드 서버에 저장하는 일까지는 일반적인 웹 개발 방식을 이용하면 어렵지 않게 만들어 낼 수 있다.

그렇다면 해당 모델 데이터를 이용해서 실제 동영상을 만들어 내려면 어떻게 하면 될까? 좀더 상세하게 요구사항을 분석해보자.

## 템플릿 기반 동영상 편집 & 생성 요구사항 분석

  1. 타임라인에 따라 멀티 트랙(특정 시점에 여러장의 image, text들을 합성 가능) 편집을 할 수 있다.
  2. GUI 기반의 편집 툴이 있어서 템플릿을 손쉽게 제작할 수 있어야 한다.
  3. 제작된 템플릿 파일을 읽어들인 후, 텍스트 또는 이미지의 경로, 설정값 등을 프로그램으로 자유롭게 변경 하고, 변경된 결과 파일을 사용하여 CLI (Command Line Interface)를 통해 이용해서 인코딩 할 수 있어야 한다.
  4. 리눅스, 맥, 윈도우 상에서 동일하게 실행이 가능하다.

ffmpeg 만 이용해서는 단순히 단일 트랙으로 이미지들의 슬라이드 쇼만 가능한 한계점이 존재하기때문에 위 조건을 만족시키지 못했다. 따라서 이를 만족시키는 여러 프로그램들을 찾아보다가 오픈소스로 공개된 MLT framework 기반의 영상 편집 프로그램인 샷컷(Shotcut)을 선택하게 되었다. Shotcut의 경우 템플릿 파일이 XML기반으로 저장되어 편집이 간단하고, 멀티플랫폼에서 모두 실행되며, GUI, CLI 툴도 모두 지원하고 있어서 위에서 제시한 요구사항들을 모두 만족시킨다.

## 샷컷(Shotcut) 설치 방법

Ubuntu 16.04 기준으로 샷컷(Shotcut)을 설치하는 방법을 설명하면 다음과 같다.

Shotcut에서 참고하고있는 공유 라이브러리(shared library)들을 미리 설치해준다.

    sudo apt-get update
    sudo apt-get install \
                    libsdl2.2 \
                    libgtk2.0-0 \
                    libgl1-mesa-glx \
                    libxml-libxslt-perl \
                    libgomp1 \
                    libsamplerate0 \
                    libjack0 \
                    libltdl7 \
                    libtheora0 \
                    libxv1 \
                    libxcb-shape0
    

Ubuntu 16.04의 경우 이미 `snap` 이 설치되어 있기때문에 아래 명령어를 이용하면 Shotcut 어플리케이션이 바로 설치가 완료된다.

    sudo snap install shotcut --classic
    

위 명령어를 실행하면 Shotcut 어플리케이션이 `/snap/shotcut/current/Shotcut.app` 경로에 설치된다.

Linux에 X Windows 기반의UI가 존재한다면, 설치된 shotcut을 실행하면 GUI기반의 편집도 가능하다. GUI 툴을 이용해서 템플릿을 만드는 것은 꼭 리눅스에서 할 필요는 없으니 맥이나 윈도우에서 해도 된다.

하지만 이글의 목적은 CLI 기반으로 동영상 생성을 자동화를 하는것이기 때문에 shotcut에 포함된 CLI tool 중에 하나인 `qmelt` 명령어를 이용하여 GUI없이 template.mlt 파일로 부터 동영상을 만들어 내는 방법에 집중할 것이다.

참고: snap을 통해 설치된 qmelt 명령어의 경로는 다음과 같다.

    /snap/shotcut/current/Shotcut.app/qmelt
    

만약 Linux에 X Windows가 설치되어있지 않거나, AWS EC2 서버처럼 그래픽 디바이스가 없는 경우 `qmelt`를 실행하게되면 아래와 같은 에러와 함께 실행이 불가능하다.

> QXcbConnection: Could not connect to display
> 
> Aborted (core dumped) 

qmelt의 경우 내부적으로 Qt 기반의 Renderer를 이용하다보니 그래픽 장치를 관련된 기능을 로드하려다가 실패하는것으로 보인다. 따라서 이를 해결하기 위해서는 xvfb (X virtual framebuffer) 라는 프로그램을 추가로 설치하면 된다.

    sudo apt-get install xvfb
    

설치된 후에 `qmelt`를 직접 실행하는 대신 `xvfb-run` 명령어를 통해 간접적으로 실행하면 정상적으로 실행됨을 확인 할 수 있다.

    xvfb-run -a /snap/shotcut/current/Shotcut.app/qmelt
    

  * 참고: `qmelt`와 `melt` 두가지 명령어는 무슨 차이인가요? 
      * Qt기반의 `qmelt`와 GTK 기반의 `melt` 두가지가 있는데, Shotcut에서는 `qmelt` 사용을 권장하고 있다. GTK기반의 `melt`의 경우 Qt가 안되는 상황에서 fallback으로 사용하도록 만들어졌는데 텍스트 필터에 포함된 물결 문자 (Tilde, &#8220;~&#8221;) 와 같은 특정 문자를 제대로 렌더링하지 못하는 문제 등이 존재하고, 그 외에도 불규칙한 화면 깜빡임 등 불안정한 요소들이 존재한다. 때문에 가능하다면 `qmelt`를 사용하는것을 추천한다.

## 템플릿 만들기

  * GUI 툴을 이용해서 원하는 형태로 타임라인과 리소스들을 구성하여 저장하면 확장자가 mlt인 프로젝트 파일이 생긴다.
  * mlt파일은 XML 형태로 되어있기때문에 이 파일을 파싱한 후, 문서 내부의 특정 image나 text들을 원하는 값으로 치환하거나, 아이템들을 추가/삭제 할 수 있다.
  * 이렇게 변경된 최종 mlt 파일을 이용하여 melt 명령어로 인코딩하여 동영상을 생성 할 수 있다.

### mlt 파일 태그 설명

mlt 파일에서 사용되는 주요 태그들을 아래와같이 간단히 정리해 보았다.

  * `producer`: 이미지, 컬러 등 각 트랙안에 배치되는 &#8220;리소스&#8221; 라고 보면 된다. 
      * 다양한 속성값을 지정이 가능하다.
      * 텍스트 필터나 사이즈 변경, 특수효과 등을 줄 수 있는 필터를 지정하면 `filter`가 이곳에 child 태그로 추가된다.
  * `playlist`: 타임라인에 배치되는 &#8220;트랙&#8221;을 의미한다 
      * `entry` 태그에는 연결된 `producer`를 지정하게되고 트랙 안에서 몇초부터 몇초까지 지속되는지를 명시한다.
  * `tractor`: 트랙들간의 레이어 순서와 mixing 방식을 정의한다.

## qmelt 명령어 사용법

아래 명령어를 통해 template.mlt 파일을 동영상으로 인코딩 할 수 있다.

    /snap/shotcut/current/Shotcut.app/qmelt -verbose -progress2 -abort ~/template.mlt
    

맥 OS의 경우 아래 경로에 실행파일이 존재한다.

    /Applications/Shotcut.app/Contents/MacOS/qmelt
    

단순하게 mlt파일을 매개변수로 넣을경우 GUI mlt viewer로 영상이 렌더링된다. 실제 영상 파일로 출력하려면 mlt파일을 열어서 `profile` 태그 다음에 `consumer` 태그를 넣어주어야 한다. 다음 내용을 참고하자. consumer 태그의 attribute 설정값 들은 Shotcut GUI 어플리케이션에서 미리 설정해 둔 값으로 넣으면 되는데 이 값을 찾는 트릭이 존재한다.

    <consumer mlt_service="avformat" aspect="1" preset="fast" f="mp4" g="15" bf="2" vcodec="libx264" ab="384k" rescale="bilinear" crf="21" target="/Users/yjiq150/Movies/untitled.mp4" height="640" deinterlace_method="yadif" r="30" progressive="1" acodec="aac" ar="48000" width="640" real_time="-4" threads="0" movflags="+faststart" top_field_first="2"/>
    

먼저 Shotcut GUI 어플리케이션에서 원하는 export옵션을 설정한 후, 실제로 export를 실행한다. 그 후에 View 메뉴의 &#8220;Application Log&#8221; 항목을 열어서 보면 다음과 같이 파일로 영상 렌더링을 하는 log를 볼 수 있다. 여기에 나오는 &#8220;/var/folders/zn/dgry6c5s27zb6ry488lhjdz80000gn/T/shotcut-fs8617.mlt&#8221; 이 mlt 파일이 실제 파일로 영상 렌더링을 지시하는 임시 파일인데, 이 파일을 열어서 살펴보면 우리가 설정했던 export 옵션을 기반으로 생성된 consumer 태그가 존재하는것을 확인 할 수 있으니 이 값을 복사해서 사용하면 된다.

    [Debug  ] <MeltJob::start> "/Applications/Shotcut.app/Contents/MacOS/qmelt" ("-verbose", "-progress2", "-abort", "/var/folders/zn/dgry6c5s27zb6ry488lhjdz80000gn/T/shotcut-fs8617.mlt"
    

## Ubuntu에 한글 폰트 설치 하기

Shotcut에서 text filter에 한글 text 입력을 해놓은 경우 인코딩을 해보면 글자가 깨지는 경우가 있다. 이는 text filter에 지정한 폰트가 우분투에 설치되어 있지 않거나, 지정한 폰트가 한글을 지원하지 않는 경우에 발생한다. 이를 해결하기 위해 아래 안내에 따라 한글 폰트를 따로 설치해주면 된다.

### 표준 폰트 경로

우분투에서는 아래와 같은 경로에 대해서 폰트를 자동으로 읽어들인다. (해당 설정값을 좀 더 자세히 보고 싶으면 /etc/fonts/fonts.conf 파일을 열어서 살펴보면 된다.)

  * /usr/share/fonts
  * /usr/local/share/fonts
  * ~/.local/share/fonts
  * ~/.font (deprecated)

### 폰트 설치

  1. 위 경로중 원하는곳에 폰트파일을 복사한다. 
      * 전체 유저에게 적용하려면 /usr/share/fonts 또는 /usr/local/share/fonts 경로 이용
      * 현재 로그인한 유저에게만 적용하려면 ~로 시작하는 경로 이용
  2. 아래 명령어를 실행하여 시스템이 폰트를 로딩하도록 한다. 
        fc-cache -v
        
    
    `-v` 옵션을 주면 구체적으로 OS에서 규칙으로 정해진 폴더 몇몇 곳을 스캔하여 읽어들이는 것을 확인 할 수 있다.

## 참고:

18.08 까지는 이후로 텍스트를 정상적으로 렌더링이 잘되었으나, 18.09 버전부터 텍스트 렌더링이 제대로 되지 않는 문제 발생 중  
https://github.com/mltframework/shotcut/releases/tag/v18.08

## 참고 자료

  * [Shotcut][1]
  * [Ubuntu Fonts][2]

 [1]: https://shotcut.org/
 [2]: https://wiki.ubuntu.com/Fonts