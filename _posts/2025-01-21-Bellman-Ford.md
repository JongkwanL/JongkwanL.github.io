---
title: Bellman-Ford
description: >
author: Jongkwan Lee
date: 2025-01-21 21:20 +0900
categories: [Algorithm]
tags: [Bellman-Ford]
pin: false
math: true

---

## 1. Bellman-Ford 알고리즘이란?

**Bellman-Ford 알고리즘**은 음의 가중치(Negative Weight)가 포함될 수 있는 그래프에서 **한 정점으로부터 다른 모든 정점까지의 최단 경로**를 찾는 알고리즘입니다.

- **핵심 특징**:
  - 음의 가중치가 있는 그래프에서도 사용할 수 있음.
  - **단**, 음의 사이클(Negative Cycle)이 존재할 경우, “해당 사이클에 도달 가능한 경로”에 대해 최단 경로가 무의미(값이 계속 줄어듦)가 되므로, 이를 검출(Detect)할 수 있음.
- **시간 복잡도**: 
  - \( O(V \times E) \)  
  - (여기서 \( V \)는 정점 개수, \( E \)는 간선 개수)

**Dijkstra 알고리즘**과 비교했을 때, Bellman-Ford는 더 느리지만 **음의 가중치 허용 여부와 음의 사이클 검출**이라는 장점을 가집니다.


## 2. Bellman-Ford 알고리즘의 아이디어

Bellman-Ford는 크게 두 단계로 진행됩니다:

1. **거리 갱신(Relaxation) 단계**:  
   모든 간선을 최대 \( V-1 \)번 반복하며 다음을 수행합니다.
   - 각 간선 \((u, v)\)와 가중치 \(w\)에 대해,
     \[
     \text{dist}[v] \;>\; \text{dist}[u] + w
     \]
     일 경우, 
     \[
     \text{dist}[v] \;=\; \text{dist}[u] + w
     \]
     로 갱신.

   여기서 **“정점 수 - 1번 반복”**하는 이유는, 최단 경로 상의 간선 최대 개수가 \( V-1 \)개이기 때문입니다.

2. **음의 사이클(Negative Cycle) 검출**:  
   거리 갱신이 끝난 후, 모든 간선에 대해 한 번 더 확인했을 때 **더 갱신되는 정점이 있다면**, 그것은 음의 사이클이 존재한다는 의미입니다.

## 3. 예시 그래프와 동작 과정

다음 예시 그래프를 살펴봅시다. (정점을 0, 1, 2, 3, 4로 표현)

```
(총 5개 정점, 10개 간선)
간선 목록 (u -> v, w):
0 -> 1 (6)
0 -> 2 (7)
1 -> 2 (8)
1 -> 3 (5)
1 -> 4 (-4)
2 -> 3 (-3)
2 -> 4 (9)
3 -> 1 (-2)
4 -> 0 (2)
4 -> 3 (7)

예시 구조(방향 그래프):

   (0)
  /   \
(6)   (7)
 /       \
v         v
(1) -> (2)
 ^       |
 |(-2)   |(-3)
 |       v
(3)<----- 
 ^ \
 |  \ (7)
 |   v
  \-(4)
    (2) -> (0) 중간 생략 (4->0)
```

### 3.1 초기화

- **시작 정점**: 0  
- **dist** 배열을 무한대로 초기화, `dist[0] = 0`, 나머지는 `∞`로 설정

| 정점 | dist 초기값  |
|:---:|:-----------:|
| 0   | 0           |
| 1   | ∞           |
| 2   | ∞           |
| 3   | ∞           |
| 4   | ∞           |

### 3.2 거리 갱신 (V-1 = 4회 반복)

#### (1회차)

간선들을 하나씩 확인하며 갱신해 나갑니다.

1. **0 -> 1 (6)**  
   \(\text{dist}[1]\) > \(\text{dist}[0] + 6\) ⇒ ∞ > (0+6)=6 ⇒ 갱신  
   \(\text{dist}[1] = 6\)
2. **0 -> 2 (7)**  
   \(\text{dist}[2]\) > 0+7=7 ⇒ ∞ > 7 ⇒ 갱신  
   \(\text{dist}[2] = 7\)
3. **1 -> 2 (8)**  
   \(\text{dist}[2]\) = 7, \(\text{dist}[1] + 8\) = 6+8=14 ⇒ 7 > 14 ? 아님 ⇒ 갱신 안 함
4. **1 -> 3 (5)**  
   \(\text{dist}[3]\) > 6+5=11 ⇒ ∞ > 11 ⇒ 갱신  
   \(\text{dist}[3] = 11\)
5. **1 -> 4 (-4)**  
   \(\text{dist}[4]\) > 6 + (-4)=2 ⇒ ∞ > 2 ⇒ 갱신  
   \(\text{dist}[4] = 2\)
6. **2 -> 3 (-3)**  
   \(\text{dist}[3]\) = 11, \(\text{dist}[2] + (-3)\) = 7-3=4 ⇒ 11 > 4 ⇒ 갱신  
   \(\text{dist}[3] = 4\)
7. **2 -> 4 (9)**  
   \(\text{dist}[4]\) = 2, \(\text{dist}[2] + 9\) = 7+9=16 ⇒ 2 > 16 ? 아님 ⇒ 갱신 안 함
8. **3 -> 1 (-2)**  
   \(\text{dist}[1]\) = 6, \(\text{dist}[3] + (-2)\) = 4-2=2 ⇒ 6 > 2 ⇒ 갱신  
   \(\text{dist}[1] = 2\)
9. **4 -> 0 (2)**  
   \(\text{dist}[0]\) = 0, \(\text{dist}[4] + 2\) = 2+2=4 ⇒ 0 > 4 ? 아님 ⇒ 갱신 안 함
10. **4 -> 3 (7)**  
    \(\text{dist}[3]\) = 4, \(\text{dist}[4] + 7\) = 2+7=9 ⇒ 4 > 9 ? 아님 ⇒ 갱신 안 함

1회차 끝난 후 dist:

| 정점 | dist |
|:---:|:----:|
| 0   | 0    |
| 1   | 2    | (갱신됨: 원래 6이었는데, 3->1 경로로 인해 2가 됨)
| 2   | 7    |
| 3   | 4    |
| 4   | 2    |

#### (2회차)

다시 모든 간선을 확인하며 갱신 시도:

1. **0 -> 1 (6)**  
   \(\text{dist}[1] = 2\), \(\text{dist}[0] + 6 = 6\) ⇒ 2 > 6 ? 아님 ⇒ 갱신 X
2. **0 -> 2 (7)**  
   \(\text{dist}[2] = 7\), \(\text{dist}[0] + 7 = 7\) ⇒ 7 > 7 ? 아님 ⇒ 갱신 X
3. **1 -> 2 (8)**  
   \(\text{dist}[2] = 7\), \(\text{dist}[1] + 8 = 2+8=10\) ⇒ 7 > 10 ? 아님 ⇒ 갱신 X
4. **1 -> 3 (5)**  
   \(\text{dist}[3] = 4\), \(\text{dist}[1] + 5 = 2+5=7\) ⇒ 4 > 7 ? 아님 ⇒ 갱신 X
5. **1 -> 4 (-4)**  
   \(\text{dist}[4] = 2\), \(\text{dist}[1] + (-4) = 2-4=-2\) ⇒ 2 > -2 ⇒ 갱신  
   \(\text{dist}[4] = -2\)
6. **2 -> 3 (-3)**  
   \(\text{dist}[3] = 4\), \(\text{dist}[2] + (-3) = 7-3=4\) ⇒ 4 > 4 ? 아님 ⇒ 갱신 X
7. **2 -> 4 (9)**  
   \(\text{dist}[4] = -2\), \(\text{dist}[2] + 9 = 16\) ⇒ -2 > 16 ? 아님 ⇒ 갱신 X
8. **3 -> 1 (-2)**  
   \(\text{dist}[1] = 2\), \(\text{dist}[3] + (-2) = 4-2=2\) ⇒ 2 > 2 ? 아님 ⇒ 갱신 X
9. **4 -> 0 (2)**  
   \(\text{dist}[0] = 0\), \(\text{dist}[4] + 2 = -2+2=0\) ⇒ 0 > 0 ? 아님 ⇒ 갱신 X
10. **4 -> 3 (7)**  
    \(\text{dist}[3] = 4\), \(\text{dist}[4] + 7 = -2+7=5\) ⇒ 4 > 5 ? 아님 ⇒ 갱신 X

2회차 후 dist:

| 정점 | dist |
|:---:|:----:|
| 0   | 0    |
| 1   | 2    |
| 2   | 7    |
| 3   | 4    |
| 4   | -2   | (갱신됨)

#### (3회차)

간선 확인:

1. **0->1**: 2 vs 6 → 갱신 X  
2. **0->2**: 7 vs 7 → 갱신 X  
3. **1->2**: 7 vs 2+8=10 → 갱신 X  
4. **1->3**: 4 vs 2+5=7 → 갱신 X  
5. **1->4**: -2 vs 2+(-4)=-2 → 같음 → 갱신 X  
6. **2->3**: 4 vs 7+(-3)=4 → 같음 → 갱신 X  
7. **2->4**: -2 vs 7+9=16 → 갱신 X  
8. **3->1**: 2 vs 4+(-2)=2 → 같음 → 갱신 X  
9. **4->0**: 0 vs -2+2=0 → 같음 → 갱신 X  
10. **4->3**: 4 vs -2+7=5 → 갱신 X

갱신 없음 → dist 동일.

#### (4회차)

또 반복해도 동일하게 갱신 없음.

**결론**: (4회 반복 후)  
\[
\text{dist} = [0, 2, 7, 4, -2]
\]

### 3.3 음의 사이클 체크

- 위 결과에서 더 이상 갱신이 일어나지 않으므로, **음의 사이클**은 없는 것으로 결론.



## 4. Bellman-Ford 알고리즘 구현 (Go, Java, Python)

아래 예시 코드는 동일한 그래프를 사용합니다. (0에서 시작)

### 4.1 Go 예시

```go
package main

import (
    "fmt"
    "math"
)

// 간선 정보를 저장할 구조체
type Edge struct {
    u, v   int
    weight int
}

func BellmanFord(start, n int, edges []Edge) ([]int, bool) {
    // dist 배열 초기화
    dist := make([]int, n)
    for i := 0; i < n; i++ {
        dist[i] = math.MaxInt32
    }
    dist[start] = 0

    // (V-1)번 반복하며 모든 간선 확인
    for i := 0; i < n-1; i++ {
        for _, e := range edges {
            u := e.u
            v := e.v
            w := e.weight

            if dist[u] != math.MaxInt32 && dist[u]+w < dist[v] {
                dist[v] = dist[u] + w
            }
        }
    }

    // 음의 사이클 체크
    for _, e := range edges {
        u := e.u
        v := e.v
        w := e.weight

        if dist[u] != math.MaxInt32 && dist[u]+w < dist[v] {
            // 음의 사이클 존재
            return dist, true
        }
    }

    return dist, false
}

func main() {
    /*
       예시 그래프 (정점: 0,1,2,3,4), 시작 정점 = 0
       edges:
       0->1(6), 0->2(7), 1->2(8), 1->3(5), 1->4(-4),
       2->3(-3), 2->4(9), 3->1(-2), 4->0(2), 4->3(7)
    */
    edges := []Edge{
        {0, 1, 6},
        {0, 2, 7},
        {1, 2, 8},
        {1, 3, 5},
        {1, 4, -4},
        {2, 3, -3},
        {2, 4, 9},
        {3, 1, -2},
        {4, 0, 2},
        {4, 3, 7},
    }

    n := 5
    start := 0

    dist, negCycle := BellmanFord(start, n, edges)

    if negCycle {
        fmt.Println("음의 사이클이 존재합니다.")
    } else {
        fmt.Println("최단 거리 결과:")
        for i := 0; i < n; i++ {
            if dist[i] == math.MaxInt32 {
                fmt.Printf("%d -> %d : 도달 불가\n", start, i)
            } else {
                fmt.Printf("%d -> %d : %d\n", start, i, dist[i])
            }
        }
    }
}
```

- **구현 흐름**:
  1. `dist` 배열 무한대로 초기화, `dist[start] = 0`
  2. **(V-1)번** 모든 간선을 확인하여 갱신
  3. 마지막에 한 번 더 간선을 확인해 갱신 발생 시 **음의 사이클** 판정


### 4.2 Java 예시

```java
import java.util.*;

class Edge {
    int u, v, w;
    Edge(int u, int v, int w) {
        this.u = u;
        this.v = v;
        this.w = w;
    }
}

public class Main {
    public static class Result {
        int[] dist;
        boolean negativeCycle;
        public Result(int[] dist, boolean negativeCycle) {
            this.dist = dist;
            this.negativeCycle = negativeCycle;
        }
    }

    public static Result bellmanFord(int start, int n, List<Edge> edges) {
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[start] = 0;

        // (V-1)번 반복
        for(int i=0; i<n-1; i++){
            for(Edge e : edges){
                if(dist[e.u] != Integer.MAX_VALUE && dist[e.u] + e.w < dist[e.v]){
                    dist[e.v] = dist[e.u] + e.w;
                }
            }
        }

        // 음의 사이클 체크
        for(Edge e : edges){
            if(dist[e.u] != Integer.MAX_VALUE && dist[e.u] + e.w < dist[e.v]){
                // 음의 사이클 존재
                return new Result(dist, true);
            }
        }

        return new Result(dist, false);
    }

    public static void main(String[] args) {
        // 예시 그래프
        List<Edge> edges = new ArrayList<>();
        edges.add(new Edge(0, 1, 6));
        edges.add(new Edge(0, 2, 7));
        edges.add(new Edge(1, 2, 8));
        edges.add(new Edge(1, 3, 5));
        edges.add(new Edge(1, 4, -4));
        edges.add(new Edge(2, 3, -3));
        edges.add(new Edge(2, 4, 9));
        edges.add(new Edge(3, 1, -2));
        edges.add(new Edge(4, 0, 2));
        edges.add(new Edge(4, 3, 7));

        int n = 5;
        int start = 0;
        Result result = bellmanFord(start, n, edges);

        if(result.negativeCycle) {
            System.out.println("음의 사이클이 존재합니다.");
        } else {
            System.out.println("최단 거리 결과:");
            for(int i=0; i<n; i++){
                if(result.dist[i] == Integer.MAX_VALUE) {
                    System.out.println(start + " -> " + i + " : 도달 불가");
                } else {
                    System.out.println(start + " -> " + i + " : " + result.dist[i]);
                }
            }
        }
    }
}
```

- **핵심 로직**:
  - `dist` 배열을 `Integer.MAX_VALUE`로 초기화
  - (V-1)번 동안 모든 간선을 확인하며 갱신
  - 마지막에 음의 사이클 여부를 체크



### 4.3 Python 예시

```python
import sys

def bellman_ford(start, n, edges):
    INF = sys.maxsize
    dist = [INF] * n
    dist[start] = 0

    # (V-1)번 반복
    for _ in range(n-1):
        for u, v, w in edges:
            if dist[u] != INF and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # 음의 사이클 체크
    for u, v, w in edges:
        if dist[u] != INF and dist[u] + w < dist[v]:
            return dist, True  # 음의 사이클 존재

    return dist, False

if __name__ == "__main__":
    # 예시 그래프
    edges = [
        (0, 1, 6),
        (0, 2, 7),
        (1, 2, 8),
        (1, 3, 5),
        (1, 4, -4),
        (2, 3, -3),
        (2, 4, 9),
        (3, 1, -2),
        (4, 0, 2),
        (4, 3, 7)
    ]
    n = 5
    start = 0

    dist, neg_cycle = bellman_ford(start, n, edges)
    if neg_cycle:
        print("음의 사이클이 존재합니다.")
    else:
        print("최단 거리 결과:")
        for i in range(n):
            if dist[i] == sys.maxsize:
                print(f"{start} -> {i} : 도달 불가")
            else:
                print(f"{start} -> {i} : {dist[i]}")
```

- **구현 흐름**:
  1. `dist`를 `sys.maxsize`로 초기화
  2. (V-1)번 간선 순회 후 갱신
  3. 마지막에 추가 확인으로 음의 사이클 판정

## 5. 요약

1. **Bellman-Ford 알고리즘의 주용도**:
   - **음의 가중치**를 포함하는 그래프에서 최단 경로를 구해야 할 때 사용.
   - **음의 사이클** 존재 여부를 판단할 수 있음.

2. **시간 복잡도**:
   - \( O(V \times E) \).
   - 정점의 개수가 많은 대규모 그래프에서는 비효율적일 수 있음.

3. **구현 시 주의사항**:
   - (V-1)번 반복으로 거리 갱신 → 그 후 한 번 더 갱신이 일어나면 음의 사이클이 존재.
   - 음의 사이클이 발견되면, 해당 사이클에 도달 가능한 모든 정점은 최단 거리가 제대로 정의되지 않음(값이 계속 줄어듦).

4. **응용**:
   - **DAG**가 아닌 일반 그래프에서 음의 간선이 존재하는 경우.
   - 금융, 경제 모델(단기 환율 차익 등)에서 음의 사이클(차익 기회) 검출.
   - 그래프가 크지 않고 음의 가중치가 필요하면 Bellman-Ford가 주로 사용됨.


### 정리

- Bellman-Ford 알고리즘은 **(V-1)번** 모든 간선에 대해 거리를 갱신하는 방식으로 **음의 가중치**가 있는 그래프에서도 최단 경로를 찾을 수 있으며, **추가 1회** 검사로 **음의 사이클** 존재 여부도 판별할 수 있습니다.
- 시간 복잡도가 \( O(V \times E) \)이므로, 그래프가 큰 경우에는 비효율적일 수 있으나, 음의 가중치가 필요한 상황에서는 기본적인 선택지입니다. 
- 구현 흐름은 간단하나, **(V-1)번 반복 → 추가 검사**라는 틀이 중요합니다.