---
layout: post
title:  "Exponentiation by Squaring"
date:   2018-5-21
categories: [algorithm]
---

<p class="intro"><span class="dropcap">E</span>xponentiation by Squaring이란 거듭제곱을 구하는 알고리즘이다.</p>

'Exponentiation'이란 거듭제곱을 뜻하고, 'Squaring'이란 제곱을 뜻하니 'Exponentiation by Squaring'은 제곱으로 거듭제곱 구하기 정도로 해석하면 될 것 같다.

## The Naive Way

만약 2^4을 구하고 싶다면 2\*2\*2\*2를 계산하면 가능하다. 수식이 나타내는 바를 그대로 코드로 옮겼기 때문에 순진한 방법이라고 부른다. 이 방법으로는 2^10까지도 크게 문제가 없을 것 같다. 하지만 2^1000000이라면? 엄청난 시간이 걸릴 것이다. 파이썬 코드로 작성한다면 아래와 같다.

```python
def exponentiation_naive(base, exponent):
  if exponent == 0:
    return 1
  elif exponent == 1:
    return base
  result = 1
  for i in range(exponent):
    result *= base
  return result  
```

이 코드는 O(n)의 시간복잡도를 가지고 있다. (exponent - 1)번 곱셈 연산을 해야만 하기 때문이다. 버블 정렬과 같은 O(n^2) 알고리즘은 아니지만 단순 비교와 swap 메소드로 이루어진 정렬에 비해 곱셈 연산을 해야만 하는 거듭제곱은 하나하나의 연산이 큰 부담이 된다.

## Exponentiation by Squaring

이 알고리즘은 순진한 방법보다 빠르고 간단하다. 기본이 되는 아이디어는 아래와 같다.

<div style="text-align: center;"><img src="{{ '/assets/img/exponentiation.png' | prepend: site.baseurl }}" alt=""></div>

매우 간단한 방법이다.

이 방법을 써서 만약 7^100을 구한다고 해보자. 100 = 50 * 2이기 때문에 아래와 같은 등식이 성립한다.

`7^100 = (7^50)^2`

이렇게만 해도 99번 곱해야 하는 것을 50번만 곱해도 가능하게 되었다. 더 나눈다면 어떻게 될까?

`7^100 = (7^50)^2 = ((7^25)^2)^2`

연산이 더 줄어들었다. 25가 홀수라 2로 나누어 떨어지지 않는다고 해도 상관없다. (7^12)^2에 7을 1번 더 곱해주면 그만이다. 위 그림에서 even, odd로 나누어 표현한 것은 이 때문이다.

이 알고리즘을 파이썬 코드로 작성해보았다.

```python
def exponentiation_by_squaring(base, exponent):
  if exponent == 0:
    return 1
  else:
    if exponent % 2 == 1:
      return base * (exponentiation_by_squaring(base, exponent // 2) ** 2)
    else:
      return exponentiation_by_squaring(base, exponent // 2) ** 2
```

가독성은 좋지만 재귀함수를 사용했기 때문에 반복문을 사용한다면 더 빨라질 수 있을 것이다. 그래서 아래와 같이 바꿔보았다.

```python
def exponentiation_by_squaring(base, exponent):
  result = 1
  while exponent != 0:
    if exponent & 0x1:
      result *= base
    exponent >>= 1
    base *= base
  return result
```

처음 코드와 비교해서 다음과 같은 부분을 바꿨다.

1. 가능한 부분은 비트 연산자로 변경(홀수 판정, 나누기 2)
2. 지수가 짝수이든 홀수이든 지수를 나누어감에 따라 base 변수를 계속 거듭제곱함
3. 지수가 홀수일 때만 result 변수에 base 변수 곱함

이렇게 계산하면 O(log(n))의 시간복잡도로 거듭제곱의 값을 구할수 있다. 거듭제곱은 암호학에서 말하는 모듈로 연산에서 많이 사용한다고 하고 알고리즘 문제에서도 심심찮게 나온다. 확실하게 외워두면 좋을 것이다.

## 파이썬의 pow()는 어떤 알고리즘을 쓸까?

지금까지 나름 알고리즘을 공부해서 코드를 작성했지만 사실 파이썬에서는 pow()라는 빌트인 메소드가 이미 존재한다. 이 메소드도 Exponentiation by Squaring 알고리즘을 사용할까?

결론부터 말하자면, 아니다. ^^;

pow() 메소드가 어떻게 동작하는지 C 수준에서 소스를 확인해볼 수 있다. github에는 파이썬을 C로 구현한 코드를 볼 수 있는데 [cpython]에서 확인해볼 수 있다. 이 저장소를 탐색해보면 최종적으로 [longobject.c]에서 pow()의 구현 부분이 나온다.

```cpp
if (Py_SIZE(b) <= FIVEARY_CUTOFF) {
    /* Left-to-right binary exponentiation (HAC Algorithm 14.79) */
    /* http://www.cacr.math.uwaterloo.ca/hac/about/chap14.pdf    */
    for (i = Py_SIZE(b) - 1; i >= 0; --i) {
        digit bi = b->ob_digit[i];

        for (j = (digit)1 << (PyLong_SHIFT-1); j != 0; j >>= 1) {
            MULT(z, z, z);
            if (bi & j)
                MULT(z, a, z);
        }
    }
}
```

코드를 보면 파이썬은 Left-to-right exponentiation 알고리즘을 사용함을 알 수 있다. `PyLong_SHIFT` 등 코드의 세부적인 내용까지 이해할 수는 없지만 2진수 exponent를 왼쪽 비트부터 순차적으로 확인하면서 계산하고 있다. 정확한 내용은 좀 더 공부해서 다른 포스트로 만들어야겠다.

[cpython]: https://github.com/python/cpython
[longobject.c]: https://github.com/python/cpython/blob/master/Objects/longobject.c
