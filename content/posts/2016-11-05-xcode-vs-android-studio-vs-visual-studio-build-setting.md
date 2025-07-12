---
title: Xcode vs. Android Studio vs. Visual Studio 프로젝트 설정 방법 비교
author: yjiq150
type: post
date: 2016-11-05T13:20:30+00:00
url: /xcode-vs-android-studio-vs-visual-studio-build-setting/
dsq_thread_id:
  - 5280632470
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/p2p-connection-for-mobile-device/"     class="post-814"><span class="crp_title">모바일 디바이스간 P2P 연결 및 데이터 전송 방법</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/mac-app-recommendation-for-developer/"     class="post-836"><span class="crp_title">개발자를 위한 필수 맥 앱(Mac App) 10선</span></a></li><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/p2p-connection-for-mobile-device/"     class="post-814"><span class="crp_title">모바일 디바이스간 P2P 연결 및 데이터 전송 방법</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - Objective-C / Swift
  - 프로그래밍 기타

---
안드로이드, iOS, 윈도우 개발을 하는경우 플랫폼의 특성에 따라 각각의 플랫폼에 맞는 IDE를 사용할 수밖에 없다. 각각의 IDE를 사용해야 하기때문에 코드 작성을 위한 언어 공부는 그렇다 치더라도, 복잡한 빌드 설정 방법에 대해서도 많은 공부가 필요하다. 이제까지의 경험으로는 개발이 제일 쉽고 개발 환경 설정이 가장 어렵다. (하하)

간단한 단일 프로젝트의 경우 간단하게 기본 제공되는 템플릿대로 이용하면 된다. 하지만 다양한 환경에 따라 배포방식이 달라지는 경우나, 같은 코드베이스에서 약간의 코드 변형을 적용하여 미묘하게 다른 여러가지의 프로덕트를 만들어내야 하는 경우 (ex: 기능제약이 있는 무료버전 앱과 유료버전 앱을 만들어내는 경우) 에는 코드 작성도 중요하지만 프로젝트 환경 설정을 어떻게 하는지가 매우 중요하다.

안타깝게도 각 플랫폼의 특성에따라 비슷한 기능임에도 사용되고 있는 용어가 다르다보니, 자신이 익숙한 플랫폼이 아닌 경우에는 어디를 찾아서 설정을 해줘야할지 해메이기 쉽상이다. 예를들어 &#8220;프로젝트&#8221; 라는 용어가 각 플랫폼에 모두 존재하지만, 각각이 의미하는 바가 미묘하게 다르다.

이러한 혼란에서 탈출하기 위한 개발자들을 위해서 이 글은 세개의 플랫폼들 중에서 적어도 하나의 환경에 익숙한 개발자가 읽었을때 가장 도움이 되도록 작성되었다. 익숙한 플랫폼에서 빌드 설정을 하는데 사용되었던 개념에 해당하는 카운터 파트의 용어를 찾아서 비교해보고 상세한 적용 방법만 따로 익히면 된다.

다음 비교 표에 모든 내용이 요약되어있으니 상세 설명과 함께 비교해서 확인하면 된다.

| 개념                     | Visual Studio        | Xcode         | AndroidStudio  |
|:---------------------- |:-------------------- |:------------- |:-------------- |
| 최상위 구성 요소              | Solution             | Workspace     | Project        |
| 최소 구성 단위               | Project              | Project       | Module         |
| 최소 구성 단위에 대한 variation | 없음                   | Target Flavor |                |
| 빌드 설정 변경               | Build Configuration  | Configuration | Build Types    |
| variation + 빌드설정 조합    | ConfigurationManager | Scheme        | Build Variants |

## Xcode 빌드 설정 구성하기

  * Workspace는 여러개의 Project를 포함할 수 있다. 
  * Project간에 dependancy 설정이 가능하여 library project가 먼저 컴파일 된 후에 excutable project가 컴파일 되도록 하는 등의 설정이 가능하다.</p> 
  * 각 Project는 Target을 여러개 가질 수 있으며, 각 타겟당 하나의 결과물(executable, library, etc)이 생성된다. 각 Target별로 다음과 같은 설정이 가능하다.
    
      * 프로젝트 내의 특정 파일을 컴파일에서 제외/포함 
          * ex) 유/무료 버전에 대해 다른 클래스를 선택해서 컴파일
      * 프로젝트 내의 리소스를 다르게 선택 
          * ex) 유/무료 버전용 이미지 다르게 선택
      * 다른 Info.plist 적용 (이부분은 Configuration으로도 조정가능) 
          * ex) 유/무료 버전의 Bundle ID를 다르게 지정
      * 다른 Preprocessor Macro를 적용 (이부분은 Configuration으로도 조정가능)
  * Project에 대해 Configuration들을 생성할 수 있고 생성된 Configuration들은 Project내의 각 Target들에서 선택적으로 사용될 수 있다. 
      * 기본적으로 Debug, Release 두개의 Configuration이 생성되어있다.
      * 위의 기본 셋의 경우 configuration 별로 디버깅 옵션이나 최적화 옵션등이 다르게 되어있다.
      * Tip: 추가적으로 DebugBeta, DebugRC, AdhocBeta, AdhocRC, Release 등의 configuration을 생성해 두고, 하나의 앱이 Phase별로 다른 서버에 접속하게 한다던지, code signing 인증서 설정 다르게 하여 배포를 다르게 할 수 있다.
      * [<img loading="lazy" width="566" height="253" src="http://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_configuration.png" alt="xcode_configuration" class="alignnone size-full wp-image-588" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_configuration.png 566w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_configuration-300x134.png 300w" sizes="(max-width: 566px) 100vw, 566px" />][1]
  * Scheme의 경우 다른 플랫폼에는 없는 독특한 개념으로 Xcode의 빌드 액션들(Build, Run, Test, Archive)에 대해 각각 미리 어떤 configuration을 적용하고 어떤 추가 옵션을 사용할지 preset을 지정할 수 있게해준다. 
      * [<img loading="lazy" width="877" height="483" src="http://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_scheme.png" alt="xcode_scheme"  class="alignnone size-full wp-image-590" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_scheme.png 877w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_scheme-300x165.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_scheme-624x343.png 624w" sizes="(max-width: 877px) 100vw, 877px" />][2]

## Android Studio 빌드 설정 구성하기

  * Project는 여러개의 Module을 포함할 수 있다.
  * Module간에 dependancy 설정이 가능하다.
  * 각 Module은 Flavor를 여러개 설정해서 사용 가능하다. 
      * Module에는 기본적인 main 폴더의 src, res가 존재한다. 이때 해당 모듈 안에 Flavor에 이름과 동일한 폴더를 만들어서 추가적인 src, res를 넣으면 main 폴더의 src, res를 override 해서 사용하게 된다. 이는 다른 플랫폼에는 없는 Android만의 독특한 빌드 구성 방법이다.
      * ex) pro, lite 라는 각각의 Flavor를 생성한 후에 pro, lite 폴더를 만든 후 각각의 폴더 안에 이름이 동일하지만 다른 이미지를 각각 넣어서 override한다.
      * [<img loading="lazy" width="898" height="557" src="http://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_flavor.png" alt="android_studio_flavor"  class="alignnone size-full wp-image-587" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_flavor.png 898w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_flavor-300x186.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_flavor-624x387.png 624w" sizes="(max-width: 898px) 100vw, 898px" />][3]
  * 각 모듈에는 Build Type을 지정할수 있고 기본적으로 debug, release가 생성되어있다.
  * Android Studio의 Build Variant 윈도우를 통해서 어떤 flavor + 어떤 buildType 인지 선택하여 빌드를 진행 할 수 있다. 
      * [<img loading="lazy" src="http://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_build_variant.png" alt="android_studio_build_variant" width="331" height="168" class="alignnone size-full wp-image-589" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_build_variant.png 331w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_build_variant-300x152.png 300w" sizes="(max-width: 331px) 100vw, 331px" />][4] 

## Visual Stuio 빌드 설정 구성하기

  * Solution은 여러개의 Project를 포함할 수 있다. 
      * 솔루션에도 구성 옵션(Solution Configuration)이 존재한다 (Visual Studio 특징)
      * 솔루션의 구성 옵션을 여러개 만들어 두고 각 옵션별로 하위에 있는 프로젝트들을 선택적으로 컴파일 가능 (각 프로젝트로 configuration, platform을 선택하는것도 가능)
  * Project간에는 dependancy 설정이 가능하며, 특정 Project를 active project 설정하여 run/debug시에 실행될 수 있도록 한다.</p> 
  * 엑스코드, 안드로이드 스튜디오와는 달리 비주얼 스튜디오의 경우 Project에 대해 target, flavor등의 설정을 하는것이 IDE내에서는 불가능하다. 
      * 해결방법1: 공통 파일을 레퍼런스로 가리키고있는 프로젝트를 두개 각각 만들어서 프로젝트별로 조금씩 다르게 설정한다.
      * 해결방법2: MSBuild파일을 직접 수정하여 각 파일에 대해 조건식을 통한 컴파일이 가능하다.<sup id="fnref-591-MSBuild"><a href="#fn-591-MSBuild" rel="footnote">1</a></sup>
  * Project에 대해 Build Configuration을 설정가능하다. (Xcode와 거의 비슷한 방식) 
      * 기본적으로 Debug, Release 두개의 Configuration이 생성되어있다.
      * Configuration과 더불어 Platform 설정이 가능하다 (ex: x86, x64, ARM, etc)
  * 구성 관리자 (Configuration Manager) 
      * Xcode의 Scheme과 비슷한 개념이지만 약간 적용 방법이 다르다
      * Xcode의 경우 각 Build Action 별로 어떤 Configuration을 사용할지 미리 Scheme 정해놓는 방법을 사용하지만 Visual Studio의 경우 현재 활성화된 솔루션 구성(Solution Configuration) 에 따라서 각각의 프로젝트가 컴파일된다.
      * [<img loading="lazy" width="697" height="433" src="http://www.letmecompile.com/wp/wp-content/uploads/2016/11/visual_studio_solution_configuration.png" alt="visual_studio_solution_configuration"  class="alignnone size-full wp-image-586" srcset="https://www.letmecompile.com/wp/wp-content/uploads/2016/11/visual_studio_solution_configuration.png 697w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/visual_studio_solution_configuration-300x186.png 300w, https://www.letmecompile.com/wp/wp-content/uploads/2016/11/visual_studio_solution_configuration-624x387.png 624w" sizes="(max-width: 697px) 100vw, 697px" />][5]

<li id="fn-591-MSBuild">
  <p>
    http://stackoverflow.com/questions/2358495/same-source-multiple-targets-with-different-resources-visual-studio-net-2008&#160;<a href="#fnref-591-MSBuild" rev="footnote">&#8617;</a> </fn></footnotes>

 [1]: http://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_configuration.png
 [2]: http://www.letmecompile.com/wp/wp-content/uploads/2016/11/xcode_scheme.png
 [3]: http://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_flavor.png
 [4]: http://www.letmecompile.com/wp/wp-content/uploads/2016/11/android_studio_build_variant.png
 [5]: http://www.letmecompile.com/wp/wp-content/uploads/2016/11/visual_studio_solution_configuration.png