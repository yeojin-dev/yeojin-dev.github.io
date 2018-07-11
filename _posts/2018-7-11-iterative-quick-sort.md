---
layout: post
title:  "반복문으로 퀵 정렬 구현하기"
date:   2018-7-11
categories: [algorithm]
---

<p class="intro"><span class="dropcap">퀵</span> 정렬은 기본적으로 재귀함수를 이용해서 구현한다. 그런데 모든 재귀함수는 반복문으로 구현할 수 있기 때문에 스택을 이용함으로써 퀵 정렬을 반복문으로 구현하는 것이 가능하다. 재귀함수는 성능상의 문제가 있기 때문에 반복문으로 구현하면 더 좋은 퍼포먼스를 얻을 수 있다.</p>

퀵 정렬을 하기 위해서는 피봇을 계산하는 partition() 메소드가 필요하다. 이 포스트에서는 이전에 구현한 코드를 그대로 사용했다. 그 외 퀵 정렬에 대한 내용은 [이 포스트]를 참고하자.

## 스택 이용하기

이전 퀵 정렬의 코드를 다시 보자.

```python
def quick_sort(list, start, end):
    if start < end:
        pivot = partition(list, start, end)
        quick_sort(list, start, pivot - 1)
        quick_sort(list, pivot + 1, end)
    return list
```

코드를 보면 start, end 패러미터 값을 바꿔가면서 quick_sort() 메소드를 재귀적으로 호출함을 알 수 있다. 그렇다면 이 **(pivot + 1, pivot - 1이 될 수도 있는start, end 패러미터 값을 별도의 스택에 저장(push)하고 호출할 때마다 꺼내서(pop) 사용하는 것은 어떨까?** 그리고 스택에 저장한 start, end 패러미터 값이 모두 push될 때까지 partition() 메소드를 호출한다. 이렇게 구현하면 결국 재귀적으로 호출하는 것과 같은 방식으로 동작할 것이다. 그럼에도 불구하고 스택과 반복문을 이용함으로써 더 짧은 실행시간 내에 정렬을 완료할 수 있다.

한편 C/C++과 달리 파이썬의 스택은 별도의 자료구조로 구현할 필요가 없다. 기본 자료형인 list가 있기 때문이다. 스택의 push는 append() 메소드로, pop은 pop() 메소드로 사용하면 되기 때문에 별도의 인덱스를 로컬 변수로 설정할 필요가 없다.

직접 작성한 코드는 아래와 같다.

## Implementation

```python
def partition(list, start, end):
    pivot = list[start]
    left = start + 1
    right = end
    done = False
    while not done:
        while left <= right and list[left] <= pivot:
            left += 1
        while left <= right and list[right] > pivot:
            right -= 1
        if right < left:
            done = True
        else:
            list[left], list[right] = list[right], list[left]
    list[start], list[right] = list[right], list[start]  # 피봇 교환
    return right

def quick_sort_iterative(list, start, end):
    stack = []
    stack.append(start)
    stack.append(end)

    while stack:
        end = stack.pop()
        start = stack.pop()
        pivot = partition(list, start, end)

        if pivot - 1 > start:
            stack.append(start)
            stack.append(pivot - 1)

        if pivot + 1 < end:
            stack.append(pivot + 1)
            stack.append(end)

    return list
```

[이 포스트]:https://boto3.readthedocs.io/en/latest/
