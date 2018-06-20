---
layout: post
title:  "Radix Sort"
date:   2018-6-20
categories: [algorithm, python]
---

<p class="intro"><span class="dropcap">R</span>adix Sort를 번역하면 '기수 정렬'이 된다. 기수 정렬은 낮은 자리수부터 비교하여 정렬해 나간다. 기수 정렬은 정렬 속도가 매우 빠르지만 메모리 사용량이 크고 특정 자료형에서는 사용할 수 없다는 단점이 있다.</p>

기수 정렬은 가장 낮은 자리수의 숫자를 기준으로 데이터를 정렬한다. 그리고 그 다음으로 낮은 자리수의 숫자를 기준으로 정렬한다. 가장 큰 자리수까지 정렬을 완료하면 모든 데이터가 정렬되는 알고리즘이다.

이미지로 표현하면 아래와 같다.

![기수 정렬](https://ds055uzetaobb.cloudfront.net/image_optimizer/22630368cbc032ea43967b2610e73ded399e22a4.png)

이미지를 보면 정렬할 데이터는 아래와 같다. (기수 정렬을 위해 가장 낮은 자리수의 숫자를 강조했다.)

- 32<span style="color:red">**9**</span>
- 45<span style="color:red">**7**</span>
- 65<span style="color:red">**7**</span>
- 83<span style="color:red">**9**</span>
- 43<span style="color:red">**6**</span>
- 72<span style="color:red">**0**</span>
- 35<span style="color:red">**5**</span>

여기서 9, 7, 7, 9, 6, 0, 5만 정렬한다. 그러면 아래와 같은 순서가 된다. 강조된 글자만 보면 정렬되어 있는 것을 알 수 있다.

- 72<span style="color:red">**0**</span>
- 35<span style="color:red">**5**</span>
- 43<span style="color:red">**6**</span>
- 45<span style="color:red">**7**</span>
- 65<span style="color:red">**7**</span>
- 32<span style="color:red">**9**</span>
- 83<span style="color:red">**9**</span>

이번에는 10의 자리의 숫자를 기준으로 정렬한다. 이미 1의 자리를 기준으로 정렬했기 때문에 10의 자리 숫자가 같아도 정렬되어 있는 상태가 된다.

- 7<span style="color:red">**2**</span>0
- 3<span style="color:red">**2**</span>9
- 4<span style="color:red">**3**</span>6
- 8<span style="color:red">**3**</span>9
- 3<span style="color:red">**5**</span>5
- 4<span style="color:red">**5**</span>7
- 6<span style="color:red">**5**</span>7

마지막으로 100의 자리수를 기준으로 정렬한다.

- <span style="color:red">**3**</span>29
- <span style="color:red">**3**</span>55
- <span style="color:red">**4**</span>36
- <span style="color:red">**4**</span>57
- <span style="color:red">**6**</span>57
- <span style="color:red">**7**</span>20
- <span style="color:red">**8**</span>39

## Code

먼저 자리수를 비교하는 counting_sort()를 구현하였다. 정의된 데이터들은 아래와 같다.

1. list는 정렬해야하는 리스트 객체이다.

2. exp는 자리수를 의미한다. exp의 값이 1이면 1의 자리수 기준, 10이면 10의 자리수 기준이 된다.

3. output 리스트는 list의 정렬된 결과를 임시 저장하는 리스트이다. 마지막에는 output 리스트의 모든 항목을 list에 복사해야 한다.

4. count 리스트는 각 자리의 값(0~9)에 해당하는 list 리스트의 항목의 누적 개수를 의미한다. 예를 들어, exp=1인 상태에서 count[0]의 값이 1이고 count[1]의 값이 3이면 list에서 1의 자리의 값이 0인 항목이 1개, 1인 항목이 2개(count[1] - count[0])인 상태임을 말한다.

코드는 아래와 같다.

```python
def counting_sort(list, exp):
    length = len(list)
    output = [0] * length
    count = [0] * 10

    # 각 자리의 값에 해당하는 list 항목의 개수 구하기
    for i in range(length):
        index = list[i] // exp
        count[(index) % 10] += 1

    # count 리스트의 값들을 누적값으로 변경
    for i in range(1, 10):
        count[i] += count[i - 1]

    i = length - 1

    # list의 항목마다 자리의 값에 해당하는 누적값을 index로
    # output 리스트에 list의 항목 값 저장
    # list는 내림차순으로 순회
    for i in list[::-1]:
        index = i // exp
        output[count[(index) % 10] - 1] = i
        count[(index) % 10] -= 1

    for i in range(length):
        list[i] = output[i]
```

radix_sort()에서는 list에서 가장 큰 값의 자리수를 계산하고 정렬해야할 자리수를 계산해 counting_sort() 메소드의 인자로 넣어준다.

```python
def radix_sort(list):
    max_value = max(list)

    exp = 1
    while max_value // exp > 0:
        counting_sort(list, exp)
        exp *= 10

    return list
```

## 시간복잡도

기수 정렬의 코드를 보면 놀라운 점이 있는데 바로 중첩된 반복문이 나타나지 않는다는 점이다. 그래서 기수 정렬의 시간복잡도는 O(dn)이 된다. 여기서 d는 가장 큰 수의 자리수(100이라면 3)를 나타낸다.

하지만 기수 정렬은 퀵 정렬보다 Cache Miss가 많이 일어나기 때문에 실질적인 실행속도는 퀵 정렬보다 느리다. ([퀵 정렬이 빠른 이유]를 참고해보자.)

[퀵 정렬이 빠른 이유]: https://yeojin-dev.github.io/blog/quick-sort/
