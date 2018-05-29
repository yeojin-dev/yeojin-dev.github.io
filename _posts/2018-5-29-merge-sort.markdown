---
layout: post
title:  "Merge Sort"
date:   2018-5-29
categories: [algorithm, python]
---

<p class="intro"><span class="dropcap">M</span>erge Sort는 분할 정복 알고리즘을 사용하는 정렬 알고리즘이다. 퀵 정렬과 달리 최선, 최악의 경우 모두 O(nlogn)의 시간복잡도를 갖지만 별도의 메모리 공간이 필요하다.</p>

병합 정렬의 병합(Merge)이란 두 정렬된 리스트를 합쳐서 하나의 정렬된 리스트를 만드는 작업을 말한다. 병합 정렬은 리스트를 잘게 나누어 부분적으로 정렬을 수행하고 병합하는 알고리즘이다.

이미지로 표현하면 아래와 같다.

![퀵 정렬](https://upload.wikimedia.org/wikipedia/commons/thumb/c/cc/Merge-sort-example-300px.gif/250px-Merge-sort-example-300px.gif)

## Example

리스트 [9, 6, 4, 8, 1, 2, 7, 3]을 병합 정렬로 정렬한다고 가정해보자. 이를 진행순서로 정리하면 아래와 같다.

| 수행과정 | 현재 리스트 상태 |
|:--------:|:----------------:|
| 분할 | *[9, 6, 4, 8][1, 2, 7, 3]* |
| 분할 | *[9, 6][4, 8]*[1, 2, 7, 3] |
| 분할 | *[9][6][4, 8]*[1, 2, 7, 3] |
| 병합 | **[6, 9]**[4, 8][1, 2, 7, 3] |
| 분할 | [6, 9]*[4][8]*[1, 2, 7, 3] |
| 병합 | [6, 9]**[4, 8]**[1, 2, 7, 3] |
| 병합 | **[4, 6, 8, 9]**[1, 2, 7, 3] |
| 분할 | [4, 6, 8, 9]*[1, 2][7, 3]* |
| 분할 | [4, 6, 8, 9]*[1][2]*[7, 3] |
| 병합 | [4, 6, 8, 9]**[1, 2]**[7, 3] |
| 분할 | [4, 6, 8, 9][1, 2]*[7][3]* |
| 병합 | [4, 6, 8, 9][1, 2]**[3, 7]** |
| 병합 | [4, 6, 8, 9]**[1, 2, 3, 7]** |
| 병합 | **[1, 2, 3, 4, 6, 7, 8, 9]** |

## Code

```python
def merge_sort(list):
    length = len(list)
    if length > 1:
        mid = length // 2
        left_list = list[:mid]
        right_list = list[mid:]

        merge_sort(left_list)
        merge_sort(right_list)

        i = 0
        j = 0
        k = 0

        length_right = len(right_list)
        length_left = len(left_list)

        while i < length_left and j < length_right:
            if left_list[i] < right_list[j]:
                list[k] = left_list[i]
                i += 1
            else:
                list[k] = right_list[j]
                j += 1
            k += 1

        while i < length_left:
            list[k] = left_list[i]
            i += 1
            k += 1

        while j < length_right:
            list[k] = right_list[j]
            j += 1
            k += 1

    return list
```

코드가 복잡해 보이지만 자세히 들여다보면 간단하다. 먼저 재귀함수를 통해 리스트를 항목이 1개뿐인 리스트가 될 때까지 나눈다. 그리고 나서 두 리스트끼리 병합을 한다.

병합을 하는 과정은 먼저 오른쪽 리스트와 왼쪽 리스트의 항목(이들은 모두 정렬된 상태)을 비교해 가면서 작은 숫자들을 순서대로 병합된 크기의 리스트에 채워넣는다. 만약 오른쪽, 왼쪽 중 어느 한 리스트의 항목을 다 채워넣었다면 남은 리스트의 항목들을 순서대로 채워넣는다. 이것은 오른쪽, 왼쪽 리스트가 모두 정렬된 상태이기 때문에 가능하다.

## 시간복잡도

병합 정렬은 O(nlogn)의 시간복잡도를 갖는다. 퀵 정렬과 달리 병합 정렬은 최선, 최악의 상황 모두 리스트를 분할하고 병합하는 과정을 동일하게 진행하기 때문에 시간복잡도는 항상 일정하다.

퀵 정렬이 최악의 경우 O(n^2)의 시간복잡도를 갖는 것에 비해서 장점이 있다고 할 수 있다. 하지만 병합 정렬은 병합한 리스트를 저장하기 위한 별도의 저장공간(위 코드의 right_list, left_list에 해당)이 필요하다.

그리고 병합 정렬의 시간복잡도가 더 우수하지만 [퀵 정렬이 빠른 이유]를 생각해볼 때 실제 실행속도는 퀵 정렬이 더 빠르다.

[퀵 정렬이 빠른 이유]: https://yeojin-dev.github.io/blog/quick-sort/
