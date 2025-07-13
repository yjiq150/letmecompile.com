---
title: Pulumi를 이용하여 코드로 AWS 리소스 관리하기
date: 2023-09-08T14:50:19+00:00
url: /manage-aws-resources-with-pulumi-iac/
cover:
  image: "/uploads/2023/09/og-image-pulumi.jpg"
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part2/"     class="post-1014"><span class="crp_title">자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/s3-vs-backblaze-comparison/"     class="post-961"><span class="crp_title">Backblaze + 클라우드플레어를 이용한 서버 비용 절약 방법</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part3/"     class="post-1019"><span class="crp_title">자바스크립트로 크롤러 만들기 3편: 다양한 유형의 웹페이지 크롤러 만들어보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part2/"     class="post-1014"><span class="crp_title">자바스크립트로 크롤러 만들기 2편: 웹페이지 크롤링을 위한 배경 지식 알아보기</span></a></li><li><a href="https://www.letmecompile.com/javascript-crawler-tutorial-part1/"     class="post-1011"><span class="crp_title">자바스크립트로 크롤러 만들기 1편: 크롤링을 위한 크롬 개발자 도구 사용법 익히기</span></a></li><li><a href="https://www.letmecompile.com/kubernetes-nlb-nginx-ingress-update/"     class="post-931"><span class="crp_title">nginx ingress controller 무중단 업데이트하기</span></a></li><li><a href="https://www.letmecompile.com/s3-vs-backblaze-comparison/"     class="post-961"><span class="crp_title">Backblaze + 클라우드플레어를 이용한 서버 비용 절약 방법</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - DevOps
  - Server
  - AWS
tags:
  - pulumi
  - aws
  - iac
  - terraform
  - typescript
  - pulumi import
  - 풀루미
  - 테라폼

---
<div class="wp-block-jetpack-markdown">
  <p>
    AWS와 같은 클라우드 서비스를 웹 콘솔만 이용하여 사용하다보면 각 리소스의 상세 설정이 어떻게 되어있는지, 리소스들간의 관계는 어떻게 되어있는지 한눈에 파악하기가 어렵다. 특히 해당 리소스에 대해 권한을 갖고있는 여러명이 공동 작업을 하는 경우 내가 아닌 다른 사람이 어떤 설정 변경을 했는지 히스토리를 추적하는것이 불가능하다. 이때 테라폼(Terraform)이나 풀루미(Pulumi) 등의 IaC(IaC, Infrastructure as Code) 플랫폼들을 이용해서 리소스를 코드로 표현하여 관리하면 이러한 문제를 해결하는 것이 가능하다.
  </p>
  
  <p>
    클라우드 리소스를 코드 형태로 관리할 수 있게 되면 다음과 같은 이점들이 생긴다.
  </p>
  
  <ul>
    <li>
      리소스 변경 히스토리 관리
    </li>
    <li>
      리소스들 간의 관계를 코드로 파악 가능 <ul>
        <li>
          예) 특정 보안그룹(Security Group)이 적용된 EC2 인스턴스를 찾기
        </li>
      </ul>
    </li>
    <li>
      비슷한 설정을 가진 리소스들을 코드 복사/붙여넣기만으로 손쉽게 생성 가능 <ul>
        <li>
          예) 동일한 접근 권한을 가진 Static website hosting 설정이 되어있는 S3 bucket을 여러개 생성
        </li>
      </ul>
    </li>
    <li>
      클라우드 계정 변경을 해야할 때 손쉽게 신규 계정에 기존 구성을 재현 가능 <ul>
        <li>
          예) 리소스를 정의 코드가 이미 존재하기 때문에 타겟 AWS 계정을 변경한 후 해당 코드 내용을 배포
        </li>
      </ul>
    </li>
  </ul>
  
  <p>
    당장 리소스가 몇개 되지 않을때는 큰 필요성을 느끼지 못하지만, 점점 사용하는 리소스가 많아지고 팀의 규모가 커질 수록 IaC 도입했을때 얻는 장점이 더 커지게된다.
  </p>
  
  <h2>
    Why Pulumi?
  </h2>
  
  <p>
    테라폼(Terraform) 같은 경우 HashiCorp Configuration Language (HCL)라는 별도의 언어를 사용하기 때문에 해당 언어의 문법을 공부해야하는 단점이 있다. 게다가 HCL의 경우 선언적인(declartive) 특성을 가진 언어라서 자유롭게 절차적인(imperative) 로직을 작성하지 못한다. 이로인해 처음 적응하는데 시간이 걸릴뿐더러 코드를 작성할 때 공통 부분을 재활용할 수 있긴 하지만 절차적인 방식에 비하면 유연성이 많이 떨어지는 편이다.
  </p>
  
  <p>
    IaC를 위한 언어가 선언적/절차적 방식 중 어느쪽이 더 좋은지에 대한 더 자세한 논쟁은 <a href="https://news.ycombinator.com/item?id=22867289">이쪽 스레드</a>에서 볼 수 있으니 더 관심 있는 분들은 읽어보는 것을 추천한다.
  </p>
  
  <p>
    이에 반해 Python, Go, JavaScript, TypeScript, C# 등의 다양한 언어를 지원하는 Pulumi를 사용하게되면 기존에 익숙한 언어를 사용할 수 있고, IDE를 통해 자동완성의 혜택까지 받으면서 인프라 설정 코드를 작성할 수 있어서 훨씬 매력적이었다.
  </p>
  
  <p>
    Pulumi의 기본적인 <a href="https://www.pulumi.com/docs/concepts/">개념(Project/Stack/State)</a>과 처음부터 새로운 리소스를 코드로 정의해서 사용하는 방법은 <a href="https://www.pulumi.com/docs/clouds/aws/get-started/">Pulumi 공식 문서</a> 혹은 인터넷 상의 여러 튜토리얼에서 이미 잘 설명되어있다. 하지만 기존에 존재하는 리소스들을 불러와서 Pulumi로 관리하는 방법에 대해서는 간간히 설명이 있긴하지만 조금 부족한 부분이 많이 있었다. 때문에 이 글에서는 Pulumi를 사용하여 이미 존재하는 클라우드 리소스를 임포트해서 코드로 관리하는 방법 위주로 설명해보고자 한다.
  </p>
  
  <h2>
    목표: AWS RDS 업그레이드 하기
  </h2>
  
  <p>
    기존 AWS상에 이미 존재하는 RDS 인스턴스 구성은 아래와 같다.
  </p>
  
  <ul>
    <li>
      MySQL 8.0.28
    </li>
    <li>
      db.t2.micro
    </li>
  </ul>
  
  <p>
    띄워 놓은지 오래된 인스턴스라 오래된 인스턴스 클래스를 사용중인 상황이다.
  </p>
  
  <p>
    비용 절감 겸 성능 향상을 위해 이를 다음과같이 업그레이드 할 예정이다.
  </p>
  
  <ul>
    <li>
      MySQL 8.0.34
    </li>
    <li>
      db.t4g.micro
    </li>
  </ul>
  
  <p>
    기존 같았으면 AWS 웹 콘솔 UI를 통해서 설정 작업을 진행했겠지만 여기서는 Pulumi 를 통해 타입스크립트(TypeScript) 코드로 리소스 설정 변경을 관리하는 방식(IaC, Infrastructure as Code)을 사용해보려고 한다.
  </p>
  
  <h2>
    Pulumi 기본 설정
  </h2>
  
  <p>
    <a href="https://www.pulumi.com/docs/install/">Pulumi 다운로드 페이지</a>를 통해 Pulumi CLI를 먼저 설치한다.
  </p>
  
  <p>
    설치가 완료되면 새로운 폴더를 하나 만들고 Pulumi 프로젝트를 초기화 한다. aws-typescript 템플릿을 선택한 후 계속 <code>ENTER</code>를 쳐서 기본값으로 진행하거나 원하는 값을 입력한다.
  </p>
  
  <pre><code>$ mkdir infra && cd infra
$ pulumi new

Please choose a template (28/220 shown):
...
&gt; aws-typescript              A minimal AWS TypeScript Pulumi program
...
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press &lt;ENTER&gt;.
Press ^C at any time to quit.

project name: (infra)
project description: (A minimal AWS TypeScript Pulumi program)
Created project 'infra'

Please enter your desired stack name.
To create a stack in an organization, use the format &lt;org-name&gt;/&lt;stack-name&gt; (e.g. `acmecorp/dev`).
stack name: (dev)
Created stack 'dev'

aws:region: The AWS region to deploy into: (us-east-1) ap-northeast-2
Saved config
</code></pre>

<p>
  초기 설정이 완료된 후 자동으로 의존성까지 설치된다.
</p>

<p>
  <code>index.ts</code> 파일을 열어보면 아래와 같이 샘플로 S3 Bucket을 생성하는 코드가 들어있다. 이부분은 과감히 삭제한다.
</p>

<pre><code>// Create an AWS resource (S3 Bucket)
const bucket = new aws.s3.Bucket("my-bucket");

// Export the name of the bucket
export const bucketName = bucket.id;
</code></pre>

<p>
  코드로 정의된 리소스가 아직 아무것도 없지만, 지금까지 설정했던 프로젝트/스택을 Pulumi Cloud상에 싱크하여 생성하기위해 다음 명령어를 수행한다.
</p>

<pre><code>$ pulumi up

Manage your Pulumi stacks by logging in.
Run `pulumi login --help` for alternative login options.
Enter your access token from https://app.pulumi.com/account/tokens
    or hit &lt;ENTER&gt; to log in using your browser  
</code></pre>

<p>
  첫 실행인 경우 위와 같이 로그인이 필요하다는 메시지가 출력된다. 안내하는대로 <code>ENTER</code>키를 눌러 웹브라우저를 실행한 후, Pulumi 계정을 만들고 로그인을 한다.
</p>

<pre><code>Waiting for login to complete...

  Welcome to Pulumi!
  ...
Previewing update (dev)

View in Browser (Ctrl+O): https://app.pulumi.com/yjiq150/infra/dev/previews/ee002f9d-ff51-434a-a60c-1297c8f906be

     Type                 Name     Plan
 +   pulumi:pulumi:Stack  infra-dev  create

Resources:
    + 1 to create

info: There are no resources in your stack (other than the stack resource).

Do you want to perform this update?  [Use arrows to move, type to filter]
&gt; yes
</code></pre>

<p>
  로그인이 성공하면 생성될 프로젝트/스택에 대해서 안내가 나오고 실제 업데이트를 진행할 것인지를 물어본다. <code>yes</code>를 선택해서 진행하면된다.
</p>

<pre><code>     Type                 Name       Status
 +   pulumi:pulumi:Stack  infra-dev  created (0.63s)

Resources:
    + 1 created
Duration: 2s
</code></pre>

<p>
  잠시 기다리면 위처럼 스택이 성공적으로 생성된 것을 확인할 수 있다.
</p>

<p>
  여기서 잠시 <code>pulumi up</code> 명령어의 동작 원리를 좀 더 자세히 알아보자.
</p>

<p>
  <img src="/uploads/2023/09/pulumi-up.png" alt="How pulumi up works" />
</p>

<p>
  Pulumi Cloud는 현재 어떤 리소스들이 있고, 리소스들이 어떻게 설정값을 가지고 있는지 상태(state)를 저장하고있다. 예를들어 어떤 리소스에 대한 정의를 코드로 추가로 작성하고 <code>pulumi up</code> 을 실행하면 코드로 작성되어있는 내용과 Pulumi Cloud에 저장된 상태를 비교하여 변경사항을 요약하여 보여준다. (실제 리소스가 존재하는 AWS의 상태와 비교를 하는것이 아님에 주의)
</p>

<p>
  사용자가 변경사항을 확인 후에 업데이트 진행여부에 대해 <code>yes</code>를 선택하면 해당 diff 내용이 Pulumi Cloud와 실제 AWS 리소스 양쪽에 모두 반영된다.
</p>

<p>
  그림에서도 알 수 있듯이 Pulumi CLI가 작성되어있는 코드를 기반으로 로컬에서 직접 AWS 리소스를 변경한다. Pulumi Cloud에는 AWS 리소스가 어떻게 구성되어있는지에 대한 &#8216;상태&#8217;만 저장될 뿐 AWS 인증 정보등은 전혀 저장되지 않기 때문에 보안상으로도 안전하다.
</p>

<p>
  위 동작 원리에 기반하여 <code>pulumi up</code> 명령어를 실제 사용할 때 알고있어야할 중요한 포인트를 정리해보면 다음과 같다.
</p>

<ul>
  <li>
    Pulumi를 통해 새롭게 리소스를 생성하거나, 기존에 Pulumi Cloud를 통해 이미 관리중이던 리소스를 변경/삭제할때 사용된다.
  </li>
  <li>
    실제 AWS에는 존재하는 리소스이지만, Pulumi Cloud상에 상태가 저장되어있지 않은 경우 해당 리소스는 Pulumi를 통해 관리되는 리소스가 아니기 때문에 <code>pulumi up</code>에 아무런 영향을 받지 않는다. (즉, Pulumi를 도입한다고해서 모든 AWS 리소스를 Pulumi로 관리할 필요가 없고 특정 리소스들만 선택적으로 코드화하여 관리할 수 있다는 의미)
  </li>
  <li>
    Pulumi Cloud상에 저장되어있는 리소스 상태는 AWS상의 리소스 상태와 동일한 상태에서 실행해야한다.
  </li>
  <li>
    만약 그렇지 않은 상황이라면, 뒤에서 설명할 <code>pulumi refresh</code> 명령을 통해서 AWS상의 리소스 최신 상태를 불러와서 Pulumi Cloud의 상태와 싱크를 해줘야한다.
  </li>
</ul>

<p>
  이제 Pulumi 코드를 작성하기 위한 기본 설정이 완료되었으니 기존 AWS 리소스를 어떻게 Pulumi 코드로 가져올 수 있는지를 살펴보자.
</p>

<h2>
  Pulumi로 기존 AWS 리소스 설정 임포트 하기
</h2>

<p>
  AWS에는 이미 존재하는 리소스이지만 아직 Pulumi로 관리되고있는 리소스가 아니라면 <code>pulumi import</code> 명령어를 이용해서 해당 리소스와 리소스의 설정을 임포트 할 수 있다.
</p>

<p>
  <img src="/uploads/2023/09/pulumi-import.png" alt="How pulumi import works" />
</p>

<p>
  위 그림에서 볼 수 있듯이 임포트를 실행하면 AWS에 있던 리소스의 상태를 읽어와서 Pulumi Cloud에 새롭게 생성하고, 그에 해당하는 코드를 자동으로 생성해서 출력해준다. 이렇게 출력된 코드를 <code>index.ts</code>에 붙여넣어 주면 코드, Pulumi Cloud, AWS가 모두 동일한 상태로 유지되게 된다.
</p>

<p>
  <code>pulumi import</code> 명령은 <code>pulumi up</code>과는 다르게 아래와같이 명령 뒤쪽에 어떤 리소스를 가져올 것인지에 대한 인자들을 필요로한다.
</p>

<pre><code>$ pulumi import {cloud_resource_type} {pulumi_resource_name} {cloud_resource_id}

</code></pre>

<p>
  위 명령어에서 <code>{pulumi_resource_name}</code>은 리소스를 가져와서 Pulumi를 통해 관리할때 사용되는 이름을 넣어주면 된다. <code>{cloud_resource_type}</code> 과 <code>{cloud_resource_id}</code> 부분은 리소스 타입마다 어떤 값을 넣어줘야하는지가 다르기 때문에 공식 문서를 참고하는것이 좋다. <a href="https://www.pulumi.com/registry/packages/aws/api-docs/">Pulumi API 공식 문서</a>에서 먼저 임포트하고자 하는 리소스 종류(여기서는 RDS)를 먼저 찾은 후, 해당 리소스에 대한 문서에서 &#8216;import&#8217; 섹션을 찾으면 해당 리소스를 임포트하는 명령어가 항상 구체적으로 명시되어있으니 이를 참고해서 명령어를 작성하면 된다. (<a href="https://www.pulumi.com/registry/packages/aws/api-docs/rds/instance/#import">RDS 인스턴스의 import 섹션 링크</a>)
</p>

<p>
  위 가이드에따라 실제로 명령어를 작성하여 실행해 보자.
</p>

<pre><code>$ pulumi import aws:rds/instance:Instance sss-database sss-db

Previewing import (dev)

     Type                 Name          Plan       Info
     pulumi:pulumi:Stack  infra-dev
 =   └─ aws:rds:Instance  sss-database  import     3 warnings

Diagnostics:
  aws:rds:Instance (sss-database):
    warning: One or more imported inputs failed to validate. This is almost certainly a bug in the `aws` provider. The import will still proceed, but you will need to edit the generated code after copying it into your program.
    warning: aws:rds/instance:Instance resource 'sss-database' has a problem: Conflicting configuration arguments: "name": conflicts with db_name. Examine values at 'sss-database.name'.
    warning: aws:rds/instance:Instance resource 'sss-database' has a problem: Conflicting configuration arguments: "db_name": conflicts with name. Examine values at 'sss-database.dbName'.

Resources:
    = 1 to import
    1 unchanged

Do you want to perform this import?  [Use arrows to move, type to filter]
&gt; yes
</code></pre>

<p>
  위처럼 AWS에서 읽어온 리소스 설정을 Pulumi로 임포트 할 것 인지 물어보는데 <code>yes</code>를 선택해서 진행하자. 여기서 warning이 3개 출력되는데 이부분은 아래쪽에서 따로 해결할 예정이니 일단 넘어가자.
</p>

<p>
  임포트가 완료되면 아래와 같은 메시지와 임포트한 리소스에 해당하는 타입스크립트 코드가 출력된다. 이 코드를 <code>index.ts</code> 파일 안에 붙여 넣자.
</p>

<pre><code>Please copy the following code into your Pulumi application. Not doing so
will cause Pulumi to report that an update will happen on the next update command.

Please note that the imported resources are marked as protected. To destroy them
you will need to remove the `protect` option and run `pulumi update` *before*
the destroy will take effect.

import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const sss_database = new aws.rds.Instance("sss-database", {
    allocatedStorage: 20,
    autoMinorVersionUpgrade: false,
    availabilityZone: "ap-northeast-2c",
    backupRetentionPeriod: 21,
    backupWindow: "20:16-20:46",
    caCertIdentifier: "rds-ca-2019",
    dbName: "sss",
    dbSubnetGroupName: "default",
    deletionProtection: true,
    enabledCloudwatchLogsExports: [
        "error",
        "audit",
        "slowquery",
        "general",
    ],
    engine: "mysql",
    engineVersion: "8.0.28",
    identifier: "sss-db",
    instanceClass: "db.t2.micro",
    iops: 3000,
    licenseModel: "general-public-license",
    maintenanceWindow: "fri:14:22-fri:14:52",
    maxAllocatedStorage: 100,
    monitoringInterval: 60,
    monitoringRoleArn: "arn:aws:iam::242953782639:role/rds-monitoring-role",
    name: "sss",
    networkType: "IPV4",
    optionGroupName: "default:mysql-8-0",
    parameterGroupName: "mysql8-config-overrides",
    port: 3306,
    skipFinalSnapshot: true,
    storageThroughput: 125,
    storageType: "gp3",
    username: "yjiq150",
    vpcSecurityGroupIds: ["sg-66d30c0d"],
}, {
    protect: true,
});
</code></pre>

<p>
  이제 위에서 잠시 무시하고지나쳤던 warning들을 해결할 차례이다. 메시지를 보면 <code>dbName</code> 필드와 <code>name</code>필드가 서로 충돌하고있다고 나와있다. Pulumi RDS문서를 보면 <code>name</code>필드는 deprecated되었고 <code>dbName</code>을 사용하라고 안내하고있다. 해당 안내대로 코드에서 <code>name : "sss"</code> 부분을 삭제하면 된다. Pulumi를 통해서 생성되지 않은 AWS 리소스를 임포트하다보면 종종 이런 문제들이 발생하는데 지금 처럼 경고 메시지와 문서를 확인해서 코드를 다듬어주면 대부분 금방 해결된다.
</p>

<p>
  이제부터 이 리소스는 코드로 관리할 수 있게 되었기 때문에 설정을 변경하고싶다면 AWS에서 직접 변경하는 대신 이 코드를 수정한 후 <code>pulumi up</code> 을 수행하면 된다. 현 상태에서는 코드를 새로운 항목을 추가하거나 기존 항목을 삭제하지 않았기 때문에 작성된 코드와 Pulumi Cloud에 저장된 리소스 상태가 동일하니 <code>pulumi up</code>을 실행하면 변경 내용이 없다고 나와야한다. 실제로 한번 <code>pulumi up</code> 명령을 수행해보자.
</p>

<pre><code>$ pulumi up

Previewing update (dev)

     Type                 Name       Plan
     pulumi:pulumi:Stack  infra-dev

Resources:
    2 unchanged
</code></pre>

<p>
  변경사항이 없다고 정상적으로 출력되는것을 확인했다.
</p>

<h2>
  AWS 리소스 설정을 Pulumi 코드를 통해 변경하기
</h2>

<p>
  이제 원래 목표였던 MySQL 버전 변경, 인스턴스 타입 변경을 진행해볼 차례이다. <code>index.ts</code> 파일에서 아래와 같이 수정을 해보자
</p>

<ul>
  <li>
    <code>engineVersion: "8.0.28"</code> -> <code>engineVersion: "8.0.34"</code>
  </li>
  <li>
    <code>instanceClass: "db.t2.micro"</code> -> <code>instanceClass: "db.t4g.micro"</code>
  </li>
</ul>

<p>
  DB버전 업과 인스턴스 클래스 변경의 경우 데이터베이스 재시작이 필요하기때문에 다운타임이 발생할 수 밖에 없다. 이 시간을 최소화하기위해 2022년 11월 경에 출시된 <a href="https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/blue-green-deployments-overview.html">RDS 블루-그린 배포</a> 방식을 사용해볼 예정이다. 이를 위해서 코드에 아래 내용을 추가해주자.
</p>

<pre><code>blueGreenUpdate: {
  enabled: true,
},
</code></pre>

{{< adsense slot="1075484447" >}}

<p>
  이제 변경된 코드 내용을 기반으로 해당 내용을 RDS 설정에 반영하기 위해 <code>pulumi up</code>을 실행하자.
</p>

<pre><code>$ pulumi up

Previewing update (dev)

     Type                 Name          Plan       Info
     pulumi:pulumi:Stack  infra-dev
 ~   └─ aws:rds:Instance  sss-database  update     [diff: +blueGreenUpdate~engineVersion,instanceClass]

Resources:
    ~ 1 to update
    1 unchanged
    
Do you want to perform this update?  [Use arrows to move, type to filter]
  yes
&gt; no
  details
</code></pre>

<p>
  변경된 내용이 요약되서 출력된다. 여기서 <code>details</code>를 선택하면 더 자세한 변경사항들을 확인할 수 있다. 변경사항을 충분히 확인 후에 문제가 없으면 <code>yes</code>를 눌러서 업데이트를 진행한다.
</p>

<pre><code>Updating (dev)

     Type                 Name          Status       Info
     pulumi:pulumi:Stack  infra-dev     running
 ~   └─ aws:rds:Instance  sss-database  updating (23s).    [diff: +blueGreenUpdate~engineVersion,instanceClass]

</code></pre>


<p>
  AWS쪽에 리소스 업데이트가 진행되는 동안에 Pulumi CLI에서는 계속 경과 시간이 표시되면서 대기하고있게된다.
</p>

<p>
  진행이 잘 되고있는지 살펴보려고 AWS 웹 콘솔을 한번 열어보았다.
</p>

<p>
  <img src="/uploads/2023/09/rds-blue-green-deploy.png" alt="RDS blue/green deployment" />
</p>

<p>
  위 그림과 같이 기존 &#8216;블루&#8217; 와 동일한 스펙의 그린 인스턴스가 새로 생성된다. 기존의 인스턴스인 &#8216;블루&#8217;가 계속 정상적으로 요청을 처리하고있는 동안, &#8216;그린&#8217; 인스턴스에는 RDS에 요청한 설정 변경들이 진행된다. 설정이 완료되면 블루/그린 인스턴스 전환을 통해 신규 인스턴스가 &#8216;블루&#8217;로 변경되어 트래픽을 받게되고 &#8216;그린&#8217; 인스턴스는 종료된다. 웹 콘솔에서 직접 블루/그린 배포를 하는 경우 방금 말한 전환 및 그린 인스턴스 종료 과정을 단계별로 수동 진행해야하지만, Pulumi로 진행한 경우에는 이 프로세스가 모두 자동화되어있기 때문에 매우 편리하다.
</p>

<p>
  모든 변경이 완료되면 터미널에 총 수행 시간이 표시되면서 Pulumi CLI가 종료되는 것을 확인할 수 있다.
</p>

<h2>
  웹 콘솔을 통해 변경한 AWS 리소스 설정을 Pulumi에 반영하기
</h2>

<p>
  앞선 내용에서 코드를 변경하고 이를 통해서 AWS 리소스 설정을 변경하는 과정을 충분히 살펴보았다. 그런데 이미 Pulumi를 통해 관리되고 있는 AWS 리소스를 코드가 아닌 AWS 웹 콘솔 혹은 AWS CLI를 통해서 리소스를 직접 변경한 경우에는 어떻게 되는걸까?
</p>

<p>
  이 경우에는 Pulumi Cloud에 저장된 상태 정보와 AWS 리소스의 상태가 틀어졌기때문에 이 정보를 동일하게 싱크해줘야 Pulumi가 정상적으로 동작하게 된다. 이러한 상황을 위해 <code>pulumi refresh</code> 명령어가 존재한다.
</p>

<p>
  <img src="/uploads/2023/09/pulumi-refresh.png" alt="How pulumi refresh works" />
</p>

<p>
  <code>import</code> 명령어와 비슷하지만 <code>import</code>의 경우에는 Pulumi Cloud에 특정 AWS 리소스의 상태가 존재하지 않을때 해당 리소스를 Pulumi Cloud에 생성하기 위해서 사용했다. 하지만 지금 가정한 상황에서는 이미 Pulumi Cloud에 특정 AWS 리소스의 상태가 존재하고 있는 상황에서 이 상태값을, 실제 AWS 리소스의 상태와 동일하게 업데이트 하는것이 목표이기 때문에 <code>import</code>가 아닌 <code>refresh</code>라는 별도의 명령어를 사용하면 된다.
</p>

<p>
  이렇게 Pulumi Cloud의 상태값을 업데이트 한 이후에는 업데이트된 내용의 diff를 보고 최종적으로 코드를 직접 수정해줘야한다. 코드를 수정해주지 않으면 Pulumi Cloud, AWS 간의 상태는 통일되었는데 코드 상태만 다른 상태가 된다. 때문에 다음번에 <code>pulumi up</code>을 수행하면 <code>refresh</code>때 업데이트된 diff 내용을 반대로 다시 업데이트 하려고 하기 때문에 주의해야한다.
</p>

<h2>
  리소스 여러개를 한번에 임포트하기
</h2>

<p>
  앞서 소개했던 <code>pulumi import</code> 명령어의 경우 한번에 하나의 리소스밖에 가져오지 못하는 단점이 있다. 때문에 기존 AWS를 통해 많은 리소스를 생성해서 운영 중이지만 기존에 IaC를 사용하고있지 않았다면 IaC도입을 할 때 임포트해야할 리소스의 숫자가 너무 많아서 부담스러울 수 있다.
</p>

<p>
  다행히도 Pulumi에서는 json 파일로부터 여러개의 리소스 정보를 받아와서 한번에 임포트(bulk import)하는 다음과 같은 명령어를 지원한다.
</p>

<pre><code>$ pulumi import -f resource.json
</code></pre>

<p>
  json 파일은 다음과 같은 형식으로 작성하면 된다.
</p>

<pre><code>{
  "resources": [
    {
      "type": "aws:ec2/instance:Instance",
      "name": "sss_web_server-i-07b6e8e6b51fe9de3",
      "id": "i-07b6e8e6b51fe9de3"
    },
    ...
  ]
}
</code></pre>

<ul>
  <li>
    <a href="https://www.pulumi.com/learn/importing/bulk-importing/">참고1: Importing in Bulk</a>
  </li>
  <li>
    <a href="https://www.pulumi.com/blog/automating-pulumi-import-to-bring-manually-created-resources-into-iac">참고2: Automating Pulumi Import with Manually Created Resources</a>
  </li>
</ul>

<p>
  그렇다면 위의 json 파일은 어떻게 생성할 수 있을까? 리소스의 숫자가 많다면 해당 내용을 직접 작성하는 것 또한 쉽지 않기때문에 자동화가 필요하다. 위의 참고2 글에서 파이썬으로 작성된 <a href="https://github.com/pulumi/pulumi-import-aws-account-scraper">pulumi-import-aws-account-scraper</a> 라는 스크립트를 이용하면 EC2 관련 리소스들을 조회하여 json 파일로 출력해줘서 매우 편리하다. 하지만 샘플 코드라서 그런가 EC2만 지원하고 S3 등의 다른 리소스는 지원하지 않고있다.
</p>

<p>
  필자의 경우 S3를 많이 사용하고있었기 때문에 S3를 bulk import 가 필요했고, 기왕 Pulumi 코드를 타입스크립트로 작성하기로 결정한 김에 관련된 모든 코드들을 타입스크립트로 통일하고 싶었었다. 그래서 위의 파이썬 코드를 타입스크립트로 모두 컨버팅 한 후 S3까지 지원하도록 <a href="https://github.com/yjiq150/pulumi-aws-resource-scraper">pulumi-aws-resource-scraper</a> 스크립트를 작성해 뒀다.
</p>

<h2>
  마무리
</h2>

<p>
  이 글에서는 기존에 이미 AWS를 사용중인 상황에서 Pulumi를 도입하는 과정에서 발생할 수 있는 궁금증들과 주요 사용 시나리오들을 전반적으로 설명해보았다. 새로운 기술을 도입하는 과정은 항상 간단하지 않고 공부와 시행착오가 필요하지만 이렇게 하나 둘씩 기존 리소스들을 Pulumi를 통해 코드로 관리할 수 있게 바꿔두면 장기적으로 인프라 관리에 소요되는 시간을 줄일 수 있고 훨씬 더 안정적이고 편리하게 인프라를 사용할 수 있게될거라 확신한다. 다들 보고만 있지 말고 이 기회에 IaC를 도입하는 도전을 해보는 것을 추천한다.
</p>
</div>
