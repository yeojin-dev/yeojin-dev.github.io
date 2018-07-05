---
layout: post
title:  "알고리즘 문제풀이 - Ugly Number"
date:   2018-7-5
categories: [algorithm]
---

<p class="intro"><span class="dropcap">코</span>딩 도장에 있는 Ugly Number 문제를 해결했다. Ugly Number의 조건을 검사하되 동적 프로그래밍 기법으로 시간복잡도를 낮출 수 있었다.</p>

도전한 문제는 [여기]에서 볼 수 있다.

## The Naive Way

Ugly Number가 맞는지 아닌지 확인하는 방법은 간단하다.

1. 2로 더 이상 나눌 수 없을 때까지 나누고 몫을 리턴한다.
2. 1의 결괏값에 대해 3으로 더 이상 나눌 수 없을 때까지 나누고 몫을 리턴한다.
3. 2의 결괏값에 대해 5로 더 이상 나눌 수 없을 때까지 나누고 몫을 리턴한다.
4. 3의 결괏값이 1이라면 Ugly Number이다.

파이썬 코드로 구현하면 아래와 같다.

```python
index = int(input())


def max_divide(num1, num2):
    while num1 % num2 == 0:
        num1 //= num2
    return num1


def is_ugly(number):
    num_div_2 = max_divide(number, 2)
    num_div_3 = max_divide(num_div_2, 3)
    result = max_divide(num_div_3, 5)

    if result == 1:
        return True
    else:
        return False


count = 0
number = 0

while count != index:
    number += 1
    if is_ugly(number):
        count += 1

print(number)
```

## Dynamic Algorithm[^1]

Ugly Number를 동적 알고리즘으로 구성해보자. 이를 위해서 Ugly Number가 갖고 있는 성질을 살펴보아야 한다.

Ugly Number는 인수로 2, 3, 5만을 갖는 수들의 수열이라고 생각해보자. 그렇다면 아래의 3개 수열도 모두 Ugly Number 수열의 부분집합이 된다.

1. 각 Ugly Number 수열의 값 * 2
2. 각 Ugly Number 수열의 값 * 3
3. 각 Ugly Number 수열의 값 * 5

여기서 원래 Ugly Number 수열은 관례적으로 1을 1번째 항으로써 취급하고, 그 다음 항부터 1~3번 수열의 값들을 하나로 모은 후 정렬하면 Ugly Number의 수열이 될 것이다.

여기서 동적 알고리즘 구현이 가능해진다.

먼저 1~3번 수열이 각각 2, 3, 5를 1번째 값으로 갖는다고 생각해보자. (그 뒤의 값들은 아직 계산, 구현되지 않은 상태이며 사실 수열로 구현하지 않아도 된다.)

새로운 Ugly Number는 1~3번 수열의 현재 값 중에서 가장 작은 값이 되는데 여기서 Ugly Number에 값을 추가시킨 수열[^2]은 다음 번째 값을 현재 값으로 갱신한다. 그리고 세 수열의 값을 비교 - Ugly Number 추가 - 해당 수열 값 갱신을 반복한다.

여기서 1~3번 수열의 다음 값은 바로 자신의 인덱스와 같은 인덱스의 Ugly Number 값 * 2, 3, 5 중 해당하는 값이 될 것이다. (각 수열의 정의를 다시 보자.) 그런데 이 경우 다음 값을 계산하기 위한 Ugly Number 값은 다른 수열에 의해 계산되었을 수 있다.

예를 들어, 2번 수열의 2번째 값은 6(2번째 Ugly Number 2 * 3)인데, 6은 Ugly Number의 6번째 값이다. 여기서 2번 수열의 2번째 값은 2번째 Ugly Number를 알아야 계산할 수 있는데, 2번째 Ugly Number는 이미 계산되어 있는 상태일 것이다.

이 부분을 동적 알고리즘으로 구현하면 된다.

파이썬 코드로 구현하면 아래와 같다. 코드가 더 잘 이해될 수도 있을 것 같다.

```python
def get_ugly_number(index):

    ugly_numbers = [0] * index
    ugly_numbers[0] = 1

    idx_ugly2 = idx_ugly3 = idx_ugly5 = 0

    next_ugly_by_2 = 2
    next_ugly_by_3 = 3
    next_ugly_by_5 = 5

    for i in range(1, index):
        ugly_numbers[i] = min(next_ugly_by_2, next_ugly_by_3, next_ugly_by_5)

        if ugly_numbers[i] == next_ugly_by_2:
            idx_ugly2 += 1
            next_ugly_by_2 = ugly_numbers[idx_ugly2] * 2

        if ugly_numbers[i] == next_ugly_by_3:
            idx_ugly3 += 1
            next_ugly_by_3 = ugly_numbers[idx_ugly3] * 3

        if ugly_numbers[i] == next_ugly_by_5:
            idx_ugly5 += 1
            next_ugly_by_5 = ugly_numbers[idx_ugly5] * 5

    return ugly_numbers[-1]


index = int(input())

print(get_ugly_number(index))
```

[여기]:http://codingdojang.com/scode/436
[^1]:이 코드는 [geeks for geeks](https://www.geeksforgeeks.org/ugly-numbers/)의 코드를 참고했다. 참고라기보다 이해 후 암기해 구현해보고 설명을 포스트에 썼다는 표현이 더 맞을 것이다.
[^2]:여기서 말하는 가장 작은 값은 중복될 수 있다. 그럴 경우 중복된 값을 가진 모든 수열이 값을 갱신한다.
