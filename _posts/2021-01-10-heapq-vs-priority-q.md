---
layout: post
title:  "[Python] 우선순위 큐 (heapq vs priority queue)"
date:   2021-01-10 15:41:11 +0900
categories: [Data Structure, Python]
comments: true
---

파이썬에서는 유용한 자료구조 라이브러리를 제공합니다. 
그 중에 하나는 우선순위 큐(priority queue)입니다.

## 우선순위 큐란?
우리는 많은 경우에서 우선순위를 만납니다.
응급실의 예를 들어보겠습니다.
다음의 환자들이 있습니다. 
스케쥴링을 이야기할 것이 아니기 때문에 치료 시간은 0으로 가정하겠습니다.

1. 5분 이내 치료해야할 사람
2. 10분 이내 치료해야할 사람
3. 2분 이내 치료해야할 사람

우리는 위의 환자의 치료 순위를 어떻게 정해야합니까?
3 -> 1 -> 2순으로 치료해야 할 것입니다. 
위의 상황을 해결해주는 자료구조가 우선순위 큐입니다.  

구현 방법은 이 글에서 다루지 않겠습니다.

## 파이썬 라이브러리
파이썬에서는 heapq, PriorityQueue로 우선순위 큐를 지원합니다.
사용법은 다음과 같습니다.

### heapq
```python
import heapq

pq = []

heapq.heappush(pq, 1)
heapq.heappush(pq, 3)
heapq.heappush(pq, 2)

heapq.heappop(pq) # 1
heapq.heappop(pq) # 2
heapq.heappop(pq) # 3
```

### PriorityQueue
```python
from queue import PriorityQueue

pq = PriorityQueue()

pq.put(1)
pq.put(3)
pq.put(2)

pq.get() # 1
pq.get() # 2
pq.get() # 3
```

다음과 같은 의문이 들 수 있습니다.
*똑같은 역할을 하는데 과연 두 라이브러리들은 무엇이 다를까?*

~~PriorityQueue는 객체이고 heapq는 여러 함수들이 들어있는 파일입니다.~~
맞는 말이긴 합니다.

정확하게 말하면 PriorityQueue은 lock을 제공하여 thread-safty class입니다.
반면에 heapq는 list를 사용하기 때문에 thread-safty class가 아닙니다.

그래서 그런지 몰라도 실행시킬 때 실행속도가 차이가 납니다. (제 추측...)

## 실행속도 비교
t2.micro에서 실험을 진행했습니다.  

```
Duration of PriorityQueue 0.980911
Duration of heapq         0.175374
```

코드
```python
from queue import PriorityQueue
import heapq
import random
from time import time

nums = [random.random() for _ in range(100000)]

priority_queue = PriorityQueue()
pq = []

start = time()
for i in range(len(nums)):
    priority_queue.put(nums[i])

for i in range(len(nums)):
    priority_queue.get()
end = time()
print(f'Duration of PriorityQueue {end - start:.6f}')


start = time()
for i in range(len(nums)):
    heapq.heappush(pq, nums[i])

for i in range(len(nums)):
    heapq.heappop(pq)
end = time()
print(f'Duration of heapq         {end - start:.6f}')
```

## 마치며
사실 이 라이브러리는 백준 [보석도둑](https://www.acmicpc.net/problem/1202)을 풀면서 알게되었습니다.
알고리즘문제를 풀때 heapq를 써야지 PriorityQueue를 쓰면 시간초과가 결렸습니다.  
다들 저와 같은 실수를 하지 않길 빌면서 이만 가보도록 하겠습니다.
~~알고리즘 어렵다!~~
