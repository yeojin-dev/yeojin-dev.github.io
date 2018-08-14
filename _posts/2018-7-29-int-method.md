---
layout: post
title:  "python int() 메소드 구현하기"
date:   2018-7-29
categories: [algorithm]
---

<p class="intro"><span class="dropcap">i</span>nt() 메소드는 숫자로 보이지만 데이터 타입이 문자열인 문자열을 정수형 데이터로 변환해주는 메소드이다. 직관적이고 간단하지만 이를 직접 구현하는 것은 약간 까다롭다. 만약 input() 메소드로 입력받은 문자열(그러나 숫자인)을 int 타입으로 바꾸려면 어떻게 해야 할까?</p>

아는 분이 직접 받아본 코딩 테스트 문제라고 한다. 실제로 충분히 있을 법한 문제인 것 같다. 이 문제를 해결하기 위해서는 ASCII 코드를 알아보아야 할 것 같다.

## ASCII code

![ASCII code](https://upload.wikimedia.org/wikipedia/commons/2/26/Ascii-codes-table.png)

ASCII 코드를 보면 각 문자마다 대응하는 십진수가 있는데 '0'부터 '9'까지 1씩 증가해나간다는 것을 알 수 있다.

그렇다면 문자 '1'을 숫자 1로 바꾸는 방법은? '1'의 ASCII 코드(49)에서 '0'의 ASCII 코드(48)을 빼면 될 것이다. 여기에 자릿수를 고려해서 코드를 작성하면 될 것 같다. 문자열의 ASCII 코드를 알아내는 파이썬 메소드는 ord()이다.

이것을 가지고 구현해보자.

## Implementation

```python
def str_to_int(num):
    result = 0
    for digit in num:
        result *= 10
        result += ord(digit) - ord('0')
    return result
```
