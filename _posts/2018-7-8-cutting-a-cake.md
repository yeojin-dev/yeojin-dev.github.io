---
layout: post
title:  "알고리즘 문제풀이 - 케이크 자르기"
date:   2018-7-8
categories: [algorithm]
---

<p class="intro"><span class="dropcap">코</span>드업의 케이크 자르기 문제에 도전해보았다. 복잡해 보이지만 아주 간단하게 풀 수 있었다.</p>

도전한 문제는 [Codeup 2628 : 케익 자르기] 문제이다.

## IDEA

그림을 다시 보자.

![참고도](http://www.codeup.kr/JudgeOnline/upload/201610/cake3.bmp)

12-53 라인과 45-99 라인이 서로 교차해서 cross가 되었다. 직관적으로 보면 이 문제의 해결 조건은 간단하다.

```어떤 라인의 숫자 2개 중 어느 하나만 다른 라인의 숫자 하나 사이에 있으면 cross 조건을 만족한다.```

중요한 것은 0개도 2개도 아닌 1개일 경우에만 cross 조건이라는 점이다.

간단한 비교로 문제를 해결할 수 있다. 아래와 같이 파이썬으로 구현했다. 파이썬은 'chained comparison'이 가능하니 더 읽기 쉬울 것이다.

## Implementation

```python
a, b = map(int, input().split())
c, d = map(int, input().split())

if a > b:
    a, b = b, a

count = 0

for i in c, d:
    if a < i < b:
        count += 1

if count == 1:
    print('cross')
else:
    print('not cross')
```

[Codeup 2628 : 케익 자르기]:http://www.codeup.kr/JudgeOnline/problem.php?id=2628
