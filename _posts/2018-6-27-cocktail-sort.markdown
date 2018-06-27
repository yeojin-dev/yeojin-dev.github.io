---
layout: post
title:  "Cocktail Sort"
date:   2018-6-27
categories: [algorithm, python]
---

<p class="intro"><span class="dropcap">C</span>ocktail Sort는 버블 정렬은 조금 더 발전시킨 알고리즘이다. 버블 정렬이 한쪽 방향으로만 최대/최솟값을 찾는 것에 비해 칵테일 정렬을 양쪽으로 최대, 최솟값을 찾는다.</p>

칵테일 정렬[^1]은 [버블 정렬]의 방식으로 최댓값을 찾아 가장 마지막 인덱스에 swap하되, 다시 1번째 인덱스로 돌아오면서 최솟값을 찾는다. 즉, 한번의 반복문에서 최대, 최솟값을 찾기 때문에 반복문이 버블 정렬의 절반만 수행된다. 물론 반복문 자체의 코드 길이는 더 길다.

이미지로 표현하면 아래와 같다.

![Cocktail Sort](https://upload.wikimedia.org/wikipedia/commons/e/ef/Sorting_shaker_sort_anim.gif)

## Code

```python

import random

def cocktail_sort(list):
    start = 0
    end = len(list) - 1
    swapped = True

    while swapped:
        swapped = False

        for i in range(start, end):
            if list[i] > list[i+1]:
                list[i], list[i+1] = list[i+1], list[i]
                swapped = True

        if not swapped:
            break

        end -= 1

        for i in range(end, start, -1):
            if list[i-1] > list[i]:
                list[i], list[i-1] = list[i-1], list[i]
                swapped = True

        if not swapped:
            break

        start += 1

    return list

num_list = list()

for i in range(20):
    num = random.randrange(500)
    num_list.append(num)

print(cocktail_sort(num_list.copy()))
```

칵테일 정렬의 특징으로는 중간에 swap을 수행했는지 하지 않았는지 flag(swapped 변수)를 통해 확인한다는 점이다. 만약 swap이 1번도 수행하지 않았다면 그 데이터는 이미 정렬되어 있음을 의미한다. 그래서 반복문을 나와서 바로 리스트를 리턴한다.

## 시간복잡도

칵테일 정렬의 시간복잡도는 [버블 정렬]과 같다. 다만 최선의 경우(이미 정렬된 경우)에는 n-1번의 비교를 수행하고 바로 리턴하기 때문에 시간복잡도가 O(n)이 된다. 정리하면 아래와 같다.

1. **칵테일 정렬의 비교** - 최선 : O(n), 최악 : O(n^2)
2. **칵테일 정렬의 swap** - 최선 : 0, 최악 : O(n^2)

[^1]: 버블 정렬처럼 '거품'이 발생하되, 좌우로 정렬하는 모습이 마치 칵테일을 흔드는 것과 같다는 비유에서 유래한 이름이다.
[버블 정렬]: https://yeojin-dev.github.io/blog/n-square-sorting/
