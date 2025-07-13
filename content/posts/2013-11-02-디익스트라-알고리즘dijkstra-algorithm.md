---
title: 디익스트라 알고리즘(Dijkstra algorithm)
date: 2013-11-01T15:04:10+00:00
url: /디익스트라-알고리즘dijkstra-algorithm/
dsq_thread_id:
  - 1926920524
crp_related_posts:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
crp_related_posts_feed:
  - '<div class="crp_related  "><h3>관련 포스트:</h3><ul><li><a href="https://www.letmecompile.com/redis-cluster-sentinel-overview/"     class="post-770"><span class="crp_title">레디스 클러스터, 센티넬 구성 및 동작 방식</span></a></li><li><a href="https://www.letmecompile.com/certificate-file-format-extensions-comparison/"     class="post-792"><span class="crp_title">인증서 파일 형식 및 확장자의 차이점 비교 설명 (Certificate file format&hellip;</span></a></li><li><a href="https://www.letmecompile.com/mysql-innodb-lock-deadlock/"     class="post-763"><span class="crp_title">MySQL InnoDB lock & deadlock 이해하기</span></a></li><li><a href="https://www.letmecompile.com/chrome-extension-with-react/"     class="post-776"><span class="crp_title">크롬 익스텐션 개발 + 리액트 적용하기</span></a></li><li><a href="https://www.letmecompile.com/api-auth-jwt-jwk-explained/"     class="post-800"><span class="crp_title">API 서버 인증을 위한 JWT와 JWK 이해하기</span></a></li></ul><div class="crp_clear"></div></div>'
categories:
  - 프로그래밍 기타
tags:
  - 디익스트라
  - 최단거리
  - 최소거리
  - 정보올림피아드
  - 디익스트라 코드
  - 디익스트라 주석

---
위키피디아에 있는 코드[^1]이 왠지 어려워서 나름대로 주석을 좀 추가하고 정리를 해봤는데, 그래도 어렵다. 그림을 그려서 설명하면 그나마 좀 괜찮은데 글로만 설명하려니 역시 쉽지 않은듯.

## 알고리즘 설명

시작노드를 설정 후 **모든 노드에 대해(메인 루프)** 아래 2가지 루프를 적용한다.

  * 서브 루프1: 현재 노드에서 도달 가능한 모든 노드 중 최소거리인 노드를 찾은 후 현재노드값을 해당 위치로 변경
  * 서브 루프2: 변경된 현재 노드까지 오는 최단거리를 알고있으므로, 이 값을 기준으로 다른 모든 노드들에 도달하는 거리를 한번씩의 덧셈으로 업데이트 한다. (현재노드까지 최단거리값 + 현재노드에서 다음노드 거리 = 출발노드에서 다음 노드까지의 최단거리)
    
          typedef int weight;
        
          const weight infinity = INT_MAX;
          const int undefined = -1;
        
          // n은 노드의 갯수,
          // w는 노드간 가중치를 나태낸 [n x n] adjacent matrix, 즉 w[i][j] 위치에 노드i, 노드j 간의 가중치가 저장되어있음
          // s는 시작 노드 번호
          // d는 시작점으로부터 각 노드까지의 거리(함수가 종료되고난 후 최단거리 결과값을 얻을 수 있음)
          void dijkstra(int n, const int w[][n], int s, weight d[])
          {
              bool is_in_S[n]; // 노드 방문 여부 표시
        
              // 초기화
              for (int v = 0; v < n; ++ v){
                  d[v] = infinity;
                  is_in_S[v] = false;
              }
        
              // 시작점 설정, 시작점에서 시작점까지의 거리는 0
              d[s] = 0;
        
              // 복잡도:(n회 루프)
              for (;;) {
                  weight min = infinity;
                  int u = undefined;
        
                  // 복잡도:(n회 루프)
                  // 도달가능한 노드중에 방문을 하지 않았으면서 최소거리인 노드를 찾고, 해당 위치를 u 로 설정한다.
                  for (int v = 0; v < n; ++ v)
                      if (!is_in_S[v] && min > d[v])
                          min = d[v], u = v;
        
                  if (u == undefined) // 추가로 도달할 수 있는 점이 없다.
                      break;
        
                  // u값이 존재할 경우 u노드를 방문한 것임.
                  // 해당 노드에 방문했다고 표기.
                  is_in_S[u] = true;
        
                  // 복잡도:(n회 루프)
                  for (int v = 0; v < n; ++ v)
                      if (w[u][v] != infinity && d[v] > d[u] + w[u][v])
                          d[v] = d[u] + w[u][v];
              }
        
              // 알고리즘 복잡도: n회 x (n회 + n회) = O(n^2)
          }
        

<div class="footnotes">
  <hr />
  
  <ol>
    <li id="fn:1">
      <p>
        http://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%81%AC%EC%8A%A4%ED%8A%B8%EB%9D%BC_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98<a href="#fnref:1" rev="footnote">&#8617;</a>
      </p>
    </li>
  </ol>
</div>