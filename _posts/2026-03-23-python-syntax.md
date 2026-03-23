---
title: 코딩 테스트 Python 문법 정리
date: 2026-03-23 00:00:00 +0900
categories: [Python]
tags: [Python, 코딩테스트, 문법]
description: 코딩 테스트에서 자주 쓰이는 Python 문법을 정리한 글입니다.
math: true
---

## 리스트

### bool 리스트와 sum()

Python에서 `True`는 `1`, `False`는 `0`으로 취급되기 때문에 bool 리스트에 `sum()`을 사용하면 `True`의 개수를 셀 수 있습니다.

```python
history = [True, False, True, True]
sum(history)  # 1 + 0 + 1 + 1 = 3
```

조건식과 함께 사용하면 특정 조건을 만족하는 원소의 개수를 한 줄로 구할 수 있습니다.

```python
nums = [1, 2, 3, 4, 5]

sum(x > 2 for x in nums)   # 3  (3, 4, 5)
sum(x % 2 == 0 for x in nums)  # 2  (2, 4)
```

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

# 올바른 2차원 리스트 초기화
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

`deque()`의 인자로 **이터러블(iterable)**을 전달해야 합니다. 반드시 리스트나 튜플로 감싸서 전달합니다.

```python
# 올바른 초기화
queue = deque([1, 2, 3])  # deque([1, 2, 3])
queue = deque((1, 2, 3))  # deque([1, 2, 3])

# 단일 값으로 초기화
queue = deque([1])        # deque([1])
```

> **warning**: `deque(1)`처럼 정수를 직접 전달하면 `TypeError`가 발생합니다. 정수는 이터러블이 아니기 때문입니다.
{: .prompt-warning }

```python
# ❌ 잘못된 초기화
queue = deque(1)   # TypeError: 'int' object is not iterable
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

---

## 입출력

### sys.stdin.readline 사용법

`input()` 대신 `sys.stdin.readline()`을 사용하면 입력 속도가 빨라져 코딩 테스트에서 시간 초과를 방지할 수 있습니다.

```python
import sys

n = int(sys.stdin.readline())
a, b = map(int, sys.stdin.readline().split())
```

> **tip**: `int()`, `split()` 등이 개행 문자(`\n`)를 알아서 처리해주기 때문에 매번 `.strip()`을 붙일 필요가 없습니다.
{: .prompt-tip }

```python
# 불필요한 strip() 없이도 동작
n = int(sys.stdin.readline())              # int()가 개행 처리
a, b = map(int, sys.stdin.readline().split())  # split()이 개행 처리

# ⚠️ strip()이 필요한 경우 — 문자열 그대로 받을 때
s = sys.stdin.readline().strip()           # "hello\n" → "hello"
```

`.strip()`은 문자열을 그대로 사용해야 할 때만 붙이면 됩니다.
