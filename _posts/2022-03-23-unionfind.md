---
layout: post
title: 다익스트라(Dijkstra) 알고리즘 추천 문제
comments: true
tags: Algorithm
minute: 1
---
<h4>유니온파인트</h4>

서로 다른 두개의 집합을 병합하는 `union`연산과 집합의 원소가 어떠한 집합에 속해있는지 판단하는 `find`연산을 지원하기에 유니온파인트라는 이름을 붙이게 되었다.

`p[]`라는 배열을 사용할 예정이다. 이 배열은 쉽게 말해서 그룹의 루트노드라고 생각하면된다. 그룹을 나눠주면서 결국에는 부모 노드까지 다 탐색을 하여 현재 노드가 같은 그룹인지 판별한다. 기존 초기화는 그룹들을 합쳐주기 이전에 부모가 `자기자신`으로 초기화 시켜야한다.

```c++
int Find(int x){
    if(x==p[x]) return x;
    return p[x]=Find(p[x]);
}
```
x의 정점이 그룹에 속하는지 안속하는지 판별하는 Find함수이다. 좀 더 자세한 구현로직은 다른 블로그를 참고해보는 것을 추천한다.

```c++
void Union(int x,int y){
    x=Find(x),y=Find(y);
    p[y]=x;
}
```

x,y의 정점을 같은 그룹으로 합쳐주는 Union함수이다.

https://www.acmicpc.net/problem/1717

https://www.acmicpc.net/problem/1976

https://www.acmicpc.net/problem/4195

https://www.acmicpc.net/problem/16562

https://www.acmicpc.net/problem/1043

https://programmers.co.kr/learn/courses/30/lessons/43162
