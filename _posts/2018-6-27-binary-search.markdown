---
layout: post
title:  "Binary Search"
date:   2018-6-27
categories: [algorithm]
---

<p class="intro"><span class="dropcap">B</span>inary Search는 정렬된 데이터에 대해서 원하는 값을 효과적으로 찾을 수 있는 알고리즘이다.</p>

알고리즘에서 정렬과 함께 가장 중요한 문제가 바로 '탐색'이다. 사실 정렬은 탐색을 위해 존재하는 것이 아닐까 하는 생각도 든다. 데이터에서 원하는 값을 찾는 탐색은 정렬되지 않은 데이터에서는 특별한 방법이 없다. 그냥 모두 다 비교해보아야(이를 Sequential Search라 부른다.) 한다. 하지만 정렬이 된 데이터에서는 훨씬 효율적인 방법으로 원하는 데이터를 탐색할 수 있다. Binary Search, 이진 탐색은 그 중 가장 간단하면서도 강력한 방법이다.

## Example

#### 시작하기 전에, 이진 탐색은 정렬된 데이터만 적용할 수 있음을 알아두자.

7개의 정수형 데이터가 있는 배열(리스트라도 상관 없다.)에서 값 50을 찾는다고 가정해보자. 이진 탐색에서는 해당 범위에서 중앙값을 찾아 비교를 한다. 그래서 중앙값을 계산하기 위해 제일 작은 인덱스와 제일 큰 인덱스를 패러미터로 설정해야만 한다. 먼저 전체 배열에서 중앙값은 35이다. 35는 50보다 작은 값이기 때문에 50은 35의 인덱스(3)보다 더 큰 인덱스를 갖고 있을 것이다. 그러므로 인덱스 4에서 6까지의 배열에 대해 이진 탐색을 재귀적으로 수행한다. 이는 중앙값과 찾고자 하는 값이 같거나(탐색 성공) 작은 인덱스(low)가 큰 인덱스(high)보다 커질 때까지(탐색 실패-찾는 값이 데이터가 없을 경우) 수행한다.

아래 그림을 보면 이해가 더 잘 될 것이다.

![이진 탐색](http://www.codenuclear.com/wp-content/uploads/2017/07/Binary_Search.jpg)

## code

```python
import random

numbers = sorted([random.randrange(1000) for i in range(20)])
value = random.choice(numbers)


def binary_search(list, value, low=0, high=0):
    if low > high:
        return False
    mid = (low + high) // 2
    if value > list[mid]:
        return binary_search(list, value, mid + 1, high)
    elif value < list[mid]:
        return binary_search(list, value, low, mid - 1)
    else:
        return mid


result = binary_search(numbers, value, low=0, high=len(numbers)-1)
print(result)
```

## 이진 탐색의 시간복잡도

이진 탐색의 시간복잡도는 조금 복잡하다.

먼저 데이터의 개수를 N이라고 할 때, 이진 탐색을 수행하면 데이터의 개수는 (1/2)N개가 된다. 그렇다면 K번 수행했을 때 데이터의 개수는 ((1/2)^K*N)개가 된다. 최악의 경우를 가정했을 때(찾고자 하는 값이 제일 앞이나 뒤에 있을 경우 - 하나의 데이터가 남은 상황) 아래의 식이 성립한다.

```
((1/2)^K*N) = 1
```

이를 K에 대해서 정리해보자. 양변에 2^K를 곱하고 정리하면,

```
2^K = N
```

위 식을 K에 대해 정리하기 위해 로그 처리하면 이진 탐색의 시간복잡도 O(logn)을 구할 수 있다.
