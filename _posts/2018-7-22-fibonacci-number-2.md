---
layout: post
title:  "피보나치 수열 (2)"
date:   2018-7-22
categories: [algorithm]
---

<p class="intro"><span class="dropcap">모</span>든 재귀함수는 반복문으로 바꿔서 코딩할 수 있다. 반복문으로 코드를 만드는 것이 재귀함수보다 OS의 개입이 적기 때문에 성능 면에서 훨씬 좋다.</p>

[지난 포스팅]에서 피보나치 수열과 다이나믹 프로그래밍에 대해서 공부했는데 조금 더 공부해보니 피보나치 수열을 반복문으로 만들 수 있음을 알았다. 이미 [반복문으로 퀵 정렬 구현하기] 포스트를 썼기 때문에 쉽게 할 수 있었다.

## Implementation

```python
def fibonacci_iterative(index):
    if index < 2:
        return index

    value = prev = 1

    for i in range(index - 2):
        value, prev = value + prev, value

    return value
```

파이썬의 swap 문법을 이용하니 피보나치 수열을 훨씬 간결하게 만들 수 있었다.    

[지난 포스팅]: https://yeojin-dev.github.io/blog/fibonacci-number-1/
[반복문으로 퀵 정렬 구현하기]: https://yeojin-dev.github.io/blog/iterative-quick-sort/
