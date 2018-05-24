---
layout: post
title:  "Quick Sort"
date:   2018-5-25
categories: [algorithm, python]
---

<p class="intro"><span class="dropcap">Q</span>uick Sort 알고리즘은 매우 빠르다. O(nlogn)의 시간복잡도를 가지면서도 CPU 캐시 히트율이 높기 때문이다.</p>

이전의 O(n^2) 알고리즘에 비해 매우 빠른 속도를 보여주는 알고리즘이다. 퀵 정렬은 정렬 대상이 되는 리스트에서 하나의 값을 Pivot으로 설정하고 Pivot의 왼쪽에는 Pivot보다 작은 값들이, Pivot의 오른쪽에는 큰 값들이 모이도록 분류한다. 이 과정에서 Pivot의 값은 정렬 후 자기 위치를 갖게 된다. 즉, 정렬된 상태가 된다. 그리고 왼쪽, 오른쪽 값들만 가진 부분 리스트에 대해 재귀적으로 퀵 정렬을 수행한다. 재귀가 끝나는 기준은 리스트의 항목이 1개 이하일 경우이다.

이미지로 표현하면 아래와 같다.

![퀵 정렬](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6a/Sorting_quicksort_anim.gif/250px-Sorting_quicksort_anim.gif)

이 포스트에서는 퀵 정렬을 파이썬으로 작성해보려고 한다. 퀵 정렬을 코드로 만들 때는 캐시를 사용할 수 있는지(부분 리스트를 별도의 메모리에 저장할 수 있는지) 여부에 따라 코드의 난이도(?)가 달라진다.

## Quick Sort with Cache

캐시를 사용할 수 있다면 아주 쉽다. Pivot을 설정하고 나머지 리스트 항목을 Pivot과 비교해 큰 값이 모인 리스트, 작은 값들이 모인 리스트를 만들어내고 재귀적으로 퀵 정렬을 수행하면 된다. 리스트 컴프리헨션을 함께 사용해 작성했다.

```python
def quick_sort_pythonic(list):
    if len(list) <= 1:
        return list
    else:
        pivot = list[0]
        less = [i for i in list[1:] if i <= pivot]
        greater = [i for i in list[1:] if i > pivot]
        return quick_sort_pythonic(less) + [pivot] + quick_sort_pythonic(greater)
```

## Quick Sort without Cache

퀵 정렬을 사용할 때 별도의 리스트 저장공간을 사용하지 않는다면 리스트 내부에서 순서를 바꿔야 한다. 이를 위해서 Pivot과 별도로 left, right 인덱스를 나타내는 변수가 필요하며 left는 오른쪽으로 탐색해가며 Pivot보다 값이 클 때, right는 왼쪽으로 탐색해가며 Pivot보다 값이 작을 때 left와 right 인덱스에 있는 리스트 항목을 서로 swap 해준다. left와 right가 서로 뒤바뀌면(right가 left보다 왼쪽에 위치하면) 그 자리에서 Pivot과 right 인덱스의 값을 swap한다. 그리고 Pivot을 기준으로 왼쪽, 오른쪽 부분만 퀵 정렬을 수행한다.

이를 위해서 퀵 정렬은 리스트 외에도 left, right의 초깃값을 설정하기 위해 시작 인덱스와 끝 인덱스를 나타낼 패러미터가 필요하다. 그리고 Pivot을 설정한 후 swap하고 새로운 Pivot을 설정하는 부분을 partition이라는 이름의 메소드로 구현했다.

코드는 아래와 같다.

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
    list[start], list[right] = list[right], list[start]  # Pivot 교환
    return right

def quick_sort(list, start, end):
    if start < end:
        pivot = partition(list, start, end)
        quick_sort(list, start, pivot - 1)
        quick_sort(list, pivot + 1, end)
    return list
```

## 시간복잡도

퀵 정렬의 시간복잡도는 최선의 경우와 최악의 경우에 따라 다르다. 퀵 소트에서 최악의 경우란 설정한 Pivot이 가장 크거나 가장 작은 값을 갖게 되는 경우-이미 정렬이 되어 있거나 역순 정렬된 상태-이다. 이 경우 Pivot을 기준으로 좌우 부분 리스트를 만들 수 없기 때문에 매번 Pivot이 아니었던 모든 항목을 비교한다. 그러므로 O(n^2)의 시간복잡도 상황이 된다.

최선의 경우는 Pivot이 항상 중간값이 되는 경우이다. 이렇게 되면 Pivot을 기준으로 리스트가 절반씩 나누어진다. 먼저 Pivot을 설정하고 left, right 인덱스를 가지고 하는 partition은 n번 수행한다. 왜냐하면 리스트의 항목이 1개 이하일 경우가 될 때까지 재귀가 발생하기 때문이다. 한편 분할이 일어나는 횟수를 k라고 하고 리스트의 항목 개수를 n이라고 할 때 둘 사이의 관계를 보면 `k = logn`의 관계가 성립한다. 그러므로 퀵 정렬은 최선의 경우 `분할 시간복잡도 * partition의 시간복잡도`의 시간복잡도를 갖는다. 즉 O(nlogn)이 된다. 

## 퀵 정렬이 빠른 또다른 이유

퀵 정렬은 O(nlogn)의 시간복잡도를 갖는 알고리즘이라 빠르기도 하지만 진짜 빠른 비결은 퀵 정렬의 메모리 참조가 지역화되어 있어서 캐시 히트율이 높기 때문이다. 여기서 캐시 히트란 CPU가 원하는 메모리가 캐시 메모리에 저장되어 있는 상태를 말하고 그 반대는 캐시 미스라고 부른다. 캐시 메모리는 레지스터 다음으로 빠른 메모리이기 때문에 캐시 히트율이 높은 프로그램은 속도가 매우 빠르다. 이것은 알고리즘과는 조금 다른 문제이다.

컴퓨터를 사용할 때 메모리 참조에 대해 학자들이 연구해보니 어떤 메모리를 참고하면 그 다음은 주변(가까이에 있는) 메모리를 참조할 확률이 높다는 특성을 발견했다. 이 특성에 따라 컴퓨터 운영체제는 캐시 메모리에 CPU가 참조한 메모리와 가까운 위치의 메모리 값들을 저장하도록 만들어졌다. 그래서 가깝거나 연속된 위치에 있는 메모리 데이터를 사용하는 프로그램은 그렇지 않은 프로그램보다 속도가 매우 빠르다. 왜냐하면 캐시 미스가 발생하면 운영체제가 캐시 메모리를 정리해야 하고 그럴 동안 내 프로그램은 연산을 멈춰야 하기 때문이다. 그만큼 실행시간이 늘어나는 것이다.

퀵 정렬을 자세히 보면 재귀가 일어나면서 리스트의 크기가 작아지고 있고 left, right 인덱스는 1씩 커지거나 작아지거나 한다. 분할이 일어날수록 인접한 인덱스의 항목을 참조하기 때문에 캐시 미스가 덜 일어난다. 퀵 정렬이 빠른 비결이 바로 여기에 있다.