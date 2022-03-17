---
layout: post
title: 플로이드 와샬(Floydwatershall) 알고리즘 추천 문제
comments: true
tags: Algorithm
minute: 1
---

<h4>플로이드 와샬이란</h4>

`모든 정점` 에서 `모든 정점`으로의 최단 경로를 구하는 알고리즘이다. `거져서 가는 정점들`  기준으로 `다이나믹 프로그래밍`을 기반으로 업데이트 한다. `음수`의 좌표에서도 사용이 가능하다.

원소 (i,j) `i로부터 j까지의 최단 경로`를 저장

```jsx
점화식 = dp[i][j] = min(dp[i][j],dp[i][n]+dp[n][j])
```

- n을 거쳐서 가는 경로와 직접 가는 경로를 비교하며 거쳐갈 수 있는 정점들을 모두 비교하여 최단 경로를 업데이트한다.


<h4>추천문제</h4>


[백준 11404번 - 플로이드](https://www.acmicpc.net/problem/11404)  


[백준 11403번 - 경로 찾기](https://www.acmicpc.net/problem/11403)  

[백준 1613번 - 역사](https://www.acmicpc.net/problem/1613)  


[백준 2458번 - 키 순서](https://www.acmicpc.net/problem/2458)  



[백준 1389번 - 케빈 베이컨의 6단계 법칙](https://www.acmicpc.net/problem/1389)



[백준 1956번 - 운동](https://www.acmicpc.net/problem/1956) 


[백준 13168번 - 내일로 여행](https://www.acmicpc.net/problem/13168) 



[백준 1507번 - 궁금한 민호](https://www.acmicpc.net/problem/1507)  


[백준 15723번 - n단 논법](https://www.acmicpc.net/problem/15723)



[프로그래머스 - 순위](https://programmers.co.kr/learn/courses/30/lessons/49191)






