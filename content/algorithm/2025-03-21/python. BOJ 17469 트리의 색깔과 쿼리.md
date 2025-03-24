---
title: python. BOJ 17469 트리의 색깔과 쿼리
draft: false
tags:
  - 알고리즘
  - python
  - 트리
  - union-find
  - small-to-large
  - 오프라인쿼리
date: 2025-03-21
---
### [문제 링크](https://www.acmicpc.net/problem/17469)

나에겐 쉽지 않은 시기들이 지나가고 있다.

그럴때일수록 작더라도 어떤 성취감이 필요하다는 생각이 든다.

그래서 플래티넘, 골드 상위 문제를 주로 푸려고 하다 발견한 문제이다.

<br>

유니온 파인드는 내가 이전에 설명한 적이 없지만, 간단히만 설명하고 넘어가겠다.

## 유니온 파인드
> 여러 노드가 있을 때, 이를 이어주는 알고리즘이다.

1. 각 노드마다 부모의 번호를 배열로 저장한다. (당연히 필요 시 map을 사용할 수 있다.)
	1. 시작 시, 스스로가 스스로의 부모라고 가정한다.
2. a, b 노드를 더할 때, (find 함수를 통해) a의 부모 ra, b의 부모 rb를 배열로부터 구한다.
3. rb를 ra에 속하도록 만들어준다.
4. 이제 b에 속하던 c라는 노드가 어디에 속하는지 검사하기 위해서 find(c)를 하면, b를 통해 a까지 도달할 수 있다.
	1. 이제 find라는 함수를 사용하면 각 노드가 어떤 부모 노드로 대표되는 집합에 속하는지 알 수 있다.

```python
parents = [i for i in range(n)]
def find(node: int) -> int:
	global parents
	if parents[node] == node:
		return node
	parents[node] = find(parents[node])
	return parents[node]

def union(a: int, b: int) -> None:
	global parents
	ra = find(a)
	rb = rind(b)
	parents[rb] = ra
```

코드는 위와 같다.

union을 할 때, ra와 rb가 같다면 이는 사이클이 발생한다는 뜻이다.

MST에서 사이클 검출용으로 유니온파인드를 사용하는 경우라면, 위 상황에 대한 적절한 에러처리를 통해 검출할 수 있다.

## 문제
> 아니 근데, 문제는 합치는게 아니라 나누는거 아닌가요?
>
> 맞다. 하지만 문제를 간단하게 하기 위해, 시간복잡도를 맞추기 위해 여러가지 테크닉이 들어간다.
>
> 1. 오프라인 쿼리
> 2. small to large 테크닉
>
> 이 그것들이다.

본 문제를 풀기위해 쿼리문을 거꾸로 적용시켜보자.

각각의 노드가 고유의 set을 갖고, 커맨드가 1일때는 합쳐주고, 2일때는 사이즈만 출력하면 되겠다.

## 오프라인 쿼리
오프라인 쿼리는 그런 것이다.

문제에서 주어진 쿼리가 실시간 쿼리가 아니라, 이미 모두 주어져있는 오프라인 쿼리인 것으로 다루는 것이다.

순서를 조작해서 문제를 훨씬 쉽게 만드는 것이다.

본 문제에서는 거꾸로 하는 것으로, 끊는 것보다 잇는 것을 훨씬 쉽게 할 수 있다.

set의 특성과 union find를 통해서 말이다.

## small to large 테크닉
두 개의 집합이 있을 때, 직접적인 원소의 이동이 있는 병합이 필요할 때 사용할 수 있을 것이다.

set의 크기를 비교해, 원소의 수가 적은 집합의 원소들을 이동시키는 것이다.

직관적으로 당연히, 사이즈가 100인 집합을 사이즈가 10인 집합에 넣는 것보다 반대가 빠를 것이다.

이 규칙을 지키기만 해도, 시간복잡도는 `O(N^2)`에서 `O(N logN)`으로  줄어든다.

최대 N개의 집합들이 매 병합마다 2배이상 크기가 커지기 때문에, 최대 병합 횟수는 logN이 되기 때문이다.

<br>

이제 이들을 적용해 문제들을 풀어보자.

## 코드
```python
import sys
from collections import deque
sys.setrecursionlimit(100010)
input = sys.stdin.readline

n, q = map(int, input().split())
parent = [int(input()) for _ in range(n-1)]
parent.insert(0, 0)
parent.insert(1, 1)
color = [int(input()) for _ in range(n)]
color.insert(0,0)

queries = [tuple(map(int, input().split())) for _ in range(q + n - 1)]

ans = deque()

sets = [set([color[i]]) for i in range(n+1)]

roots = [i for i in range(n+1)]

def find(v):
    if roots[v] == v:
        return v
    roots[v] = find(roots[v])
    return roots[v]


for cmd, node in reversed(queries):
    if cmd == 1:
        # 합치기
        # 부모 노드와 합친다. (union find)
        r_node = find(node)
        r_p_node = find(parent[node])
        roots[r_node] = r_p_node
        # 부모 노드와 자식 노드의 크기를 비교해 Small to Large 병합
        # 결과값으로는 부모set은 병합된 set을 가지고, 자식set은 빈 set을 갖도록 해준다.
        # 분기를 통해 부모셋을 항상 크게 한다.
        if len(sets[r_node]) > len(sets[r_p_node]):
            sets[r_node], sets[r_p_node] = sets[r_p_node], sets[r_node]

        while sets[r_node]:
            sets[r_p_node].add(sets[r_node].pop())
    else:
        # ans에 출력값 넣기
        r_node = find(node)
        ans.append(len(sets[r_node]))

while ans:
    print(ans.pop())
```

## 후기
- 우선 여러가지 테크닉에 대해 배울 수 있어서 좋았다.
- 막연히 알고리즘을 시작했을 때 생각했던 내용들을 보완해서 적용시키기 시작한 느낌이 든다.
	- ~~`Q. 이건 작은걸 큰쪽에 합쳐줘야 좋은거 아닌가? A. 그런 처리 안해줘도 최악의 시간을 고려하면 똑같다.` 와 같은 잘 모르던 시절 내 생각을 증명을 통해 반박해준다.~~
- 이 문제를 파이썬에서 **한번에** 통과하신 분들이 채점현황에 많던데 아래와 같은 이유로 존경한다.
	- find를 하면 최대 노드횟수만큼 거슬러 올라갈 수 있기에 recursion limit을 풀어줘야한다.
	- ***set.union(set) 을 사용하면 양 쪽 set의 크기의 합만큼의 시간이 들기에, small to large 테크닉을 전혀 사용하지 않는 것이 된다.***
	- -> 한 원소씩 직접 넣어주는 방식을 사용해야된다.
- small to large 테크닉은 시간에 관한 것일텐데, len 비교를 통한 최적화 부분을 빼면 `메모리 초과`가 난다. 이유가 뭔지 모르겠다. [해결되면 본 포스트를 업데이트 하겠다.](https://www.acmicpc.net/board/view/157815#post)