---
layout: post
title:  "O(n^2) 정렬 알고리즘들"
date:   2018-5-23
categories: [algorithm, python]
---

<p class="intro"><span class="dropcap">정</span>렬 알고리즘은 가장 기본이 되는 알고리즘이다. 정렬 알고리즘 중에서 O(n^2)의 시간복잡도를 가진 알고리즘을 알아보자.</p>

포스팅을 시작하기 전에 정렬 알고리즘은 파이썬을 기준으로 작성했기 때문에 대상이 되는 숫자들은 리스트이고 오름차순 정렬이라고 가정하였다. 순서가 내림차순이든, 정렬 기준이 숫자가 아니라 다른 기준이든 큰 차이는 없을 것이다. 결국에는 정렬하고자 하는 기준을 숫자로 변환시켜서 정렬해야 하기 때문이다. 해당 부분은 데이터 처리의 영역이다.

## Selection Sort

선택 정렬이란 전체 리스트에서 n번째로 작은 값을 찾아내서 리스트의 n번째 항목으로 이동(정확히 표현하자면 swap)시키면 된다. 이것을 1번째에서 (리스트의 전체 길이 - 1)번째까지 반복해주면 된다.

이미지로 표현하면 아래와 같다.

![선택 정렬](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b0/Selection_sort_animation.gif/250px-Selection_sort_animation.gif)

파이썬 코드로 나타내면 아래와 같다.

```python
list_raw = [9, 1, 6, 8, 4, 3, 2, 0, 5, 11, 7, -1]

# 선택 정렬
def selection_sort(list):
    length = len(list)
    for i in range(length - 1):
        min_index = i
        for j in range(i + 1, length):
            if list[min_index] > list[j]:
                min_index = j
        if min_index != i:
            list[i], list[min_index] = list[min_index], list[i]
    return list

print("selection sort")
print(selection_sort(list_raw.copy()))

# selection sort
# [-1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11]
```

선택 정렬은 비교(조건문)는 최선, 최악의 경우에도 똑같이 모두 진행하기 때문에 O(n^2)의 시간복잡도를 갖는다. 반면 swap은 최선의 경우에는 1번도 수행하지 않는다. 여기서 최선의 경우란 이미 모든 리스트가 정렬되어 있을 경우이다. 최악의 경우에는 swap이 1번씩 일어나기 때문에 O(n)의 시간복잡도를 갖는다. 최악의 경우란 최선의 경우의 반대(정반대 순서)를 말한다.

1. **선택 정렬의 비교** - 최선 : O(n^2), 최악 : O(n^2)
2. **선택 정렬의 swap** - 최선 : 0, 최악 : O(n)

## Bubble Sort

버블 정렬이란 처음부터 인접한 두 리스트 항목을 비교 후 swap해 나감으로써 가장 큰 값을 리스트의 맨 뒤로 위치시킨다. 그 후에는 가장 마지막 항목에 가장 큰 값이 위치해 있기 때문에 처음부터 (마지막 - 1)번째까지의 리스트에 대해 앞의 방법을 반복한다. 결과적으로 뒤에서 2번째 큰 값을 찾을 수 있게 된다. 이 방법을 끝까지 정렬하는 방식이 버블 정렬이다.

이미지로 표현하면 아래와 같다.

![버블 정렬](https://upload.wikimedia.org/wikipedia/commons/3/37/Bubble_sort_animation.gif)

파이썬 코드로 나타내면 아래와 같다.

```python
list_raw = [9, 1, 6, 8, 4, 3, 2, 0, 5, 11, 7, -1]

# 버블 정렬
def bubble_sort(list):
    length = len(list)
    for i in range(length, 0, -1):
        for j in range(i - 1):
            if list[j] > list[j + 1]:
                list[j], list[j + 1] = list[j + 1], list[j]
    return list

print("bubble sort")
print(bubble_sort(list_raw.copy()))

# bubble sort
# [-1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11]
```

버블 정렬은 최선의 경우 swap이 일어나지 않는다. 최악의 경우에는 매번 swap을 수행해야 한다. 비교의 경우 최선, 최악 모두 매번 수행한다. 시간복잡도로 정리하면 아래와 같다.

1. **버블 정렬의 비교** - 최선 : O(n^2), 최악 : O(n^2)
2. **버블 정렬의 swap** - 최선 : 0, 최악 : O(n^2)

## Insertion Sort

삽입 정렬이란 처음 2개 항목의 리스트만 정렬한다. 그리고 다음에는 처음부터 3번째 항목까지만 정렬한다. 앞의 2번째까지의 항목은 정렬이 되어있기 때문에 3번째 항목만 자기 위치를 찾으면 된다. 자기 위치를 찾는 방법은 현재 리스트 범위 내에서 뒤에서부터 값을 비교해나가면서 진행한다. swap의 개념이 앞의 두 정렬과는 조금 다름에 유의해야 한다.

이미지로 표현하면 아래와 같다.

![삽입 정렬](https://upload.wikimedia.org/wikipedia/commons/2/25/Insertion_sort_animation.gif)

파이썬 코드로 나타내면 아래와 같다.

```python
list_raw = [9, 1, 6, 8, 4, 3, 2, 0, 5, 11, 7, -1]

# 삽입 정렬
def insertion_sort(list):
    length = len(list)
    for i in range(1, length):
        curr_value = list[i]
        index = i
        while index > 0 and list[index - 1] > curr_value:
            list[index] = list[index - 1]
            index -= 1
        list[index] = curr_value
        print(list)
    return list

print("insertion sort")
print(insertion_sort(list_raw.copy()))

# insertion sort
# [-1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11]
```

삽입 정렬은 최선의 경우 swap이 일어나지 않는다. 최악의 경우에는 비교할 때마다 swap을 수행해야 한다. 비교의 경우 최선의 경우 n-1번만 수행(시간복잡도는 O(n))하고 최악의 경우에는 O(n^2)의 복잡도를 갖는다. 정리하면 아래와 같다.

1. **삽입 정렬의 비교** - 최선 : O(n), 최악 : O(n^2)
2. **삽입 정렬의 swap** - 최선 : 0, 최악 : O(n^2)

## 최종 정리

| 정렬 | 비교 시간복잡도 | swap 시간복잡도 |
|:-----------|:-------|:-------|
| 선택(최선) | O(n^2) | 0 |
| 선택(최악) | O(n^2) | O(n) |
| 버블(최선) | O(n^2) | 0 |
| 버블(최악) | O(n^2) | O(n^2) |
| 삽입(최선) | O(n) | 0 |
| 삽입(최악) | O(n^2) | O(n^2) |

표를 보면 O(n^2) 정렬 알고리즘 중에서 선택 정렬이 가장 낫다고 생각할 수 있다. 최악의 경우를 상정한다면 말이다. 하지만 최선의 경우에는 삽입 정렬이 가장 성능이 좋다.

## 파이썬의 sort()는 어떤 알고리즘을 쓸까?

파이썬의 빌트인 메소드인 sort()는 어떤 알고리즘을 쓸까? 파이썬은 Timsort라고 불리는 알고리즘을 사용한다. [위키백과의 설명]에 따르면 이 정렬 알고리즘은 삽입 정렬과 병합 정렬[^1]이 혼합된 알고리즘이라고 한다.

[^1]: 삽입 정렬과 달리 병합 정렬 알고리즘은 O(n^2) 정렬 알고리즘이 아니다.
[위키백과의 설명]: https://en.wikipedia.org/wiki/Timsort
