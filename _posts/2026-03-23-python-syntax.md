---
title: 코딩 테스트 Python 문법 정리
date: 2026-03-23 00:00:00 +0900
categories: [Python]
tags: [Python, 코딩테스트, 문법]
description: 코딩 테스트에서 자주 쓰이는 Python 문법을 정리한 글입니다.
math: true
---

## 리스트

### 초기화

#### 곱셈으로 리스트 초기화

`[값] * 개수` 형태로 동일한 값이 반복된 리스트를 만들 수 있습니다.

```python
[0] * 5      # [0, 0, 0, 0, 0]
[1] * 4      # [1, 1, 1, 1]
[None] * 3   # [None, None, None]
```

코딩 테스트에서는 주로 배열을 특정 값으로 초기화할 때 사용합니다.

```python
n = 5
visited = [False] * (n + 1)   # [False, False, False, False, False, False]
dist = [float('inf')] * (n + 1)  # 최단 거리 초기화
parent = [0] * (n + 1)           # 부모 노드 초기화
```

> **warning**: 2차원 리스트를 곱셈으로 초기화하면 같은 리스트 객체를 참조하게 되어 의도치 않은 동작이 발생합니다.
{: .prompt-warning }

```python
# ❌ 잘못된 2차원 리스트 초기화
graph = [[0] * 3] * 3
graph[0][0] = 1
print(graph)  # [[1, 0, 0], [1, 0, 0], [1, 0, 0]] — 모든 행이 같이 변경됨

# ✅ 올바른 2차원 리스트 초기화
graph = [[0] * 3 for _ in range(3)]
graph[0][0] = 1
print(graph)  # [[1, 0, 0], [0, 0, 0], [0, 0, 0]] — 의도한 대로 동작
```

내부 리스트를 `* n`으로 복사하면 같은 객체를 $n$번 참조하는 것이기 때문에, 2차원 이상은 반드시 **리스트 컴프리헨션**으로 초기화합니다.

---

## 큐 (Queue)

### deque 기본 사용법

Python에서 큐는 `collections` 모듈의 `deque`를 사용합니다.

```python
from collections import deque

queue = deque()

queue.append(1)     # 오른쪽에 추가 → [1]
queue.append(2)     # 오른쪽에 추가 → [1, 2]

queue.popleft()     # 왼쪽에서 제거 → 1 반환, queue = [2, 3]
queue.appendleft(0) # 왼쪽에 추가  → [0, 2, 3]
queue.pop()         # 오른쪽에서 제거 → 3 반환, queue = [0, 2]
```

### list 대신 deque를 사용하는 이유
`list.pop(0)`은 첫 번째 원소를 제거한 후 나머지 요소들을 한 칸씩 앞으로 이동시켜야 하므로 $O(N)$의 시간이 걸립니다.  
반면 `deque.popleft()`는 요소를 이동시키지 않고 앞쪽 요소만 제거하기 때문에 $O(1)$에 수행됩니다.  
따라서 큐처럼 앞에서 요소를 자주 제거해야 하는 경우에는 `deque`를 사용하는 것이 효율적입니다.

### 초기값과 함께 선언

```python
queue = deque([1, 2, 3])  # deque([1, 2, 3])
```

### BFS에서의 활용

```python
from collections import deque

def bfs(graph, start):
    visited = set([start])
    queue = deque([start])

    while queue:
        node = queue.popleft()
        print(node, end=' ')

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```
