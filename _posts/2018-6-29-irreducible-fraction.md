---
layout: post
title:  "알고리즘 문제풀이 - Irreducible Fraction"
date:   2018-6-29
categories: [algorithm]
---

<p class="intro"><span class="dropcap">I</span>rreducible Fraction(기약분수)를 구하기 위한 알고리즘 문제를 해결했다. 기약분수를 구하기 위해서는 분모, 분자의 최대공약수를 구해야 하는데 이는 유클리드 호제법을 통해 구할 수 있다.</p>

도전한 문제는 [CodeUp 2604 : 실수를 기약 분수로 변환] 문제이다.

## Euclidean Algorithm

인류 최초의 알고리즘이라고 알려진 유클리드 호제법은 최대공약수를 구하는 알고리즘이다. 유클리드 호제법은 **2개의 자연수(또는 정식) a, b에 대해서 a를 b로 나눈 나머지를 r이라 하면(단, a>b), a와 b의 최대공약수는 b와 r의 최대공약수와 같다**는 알고리즘으로 그림으로 표현하면 아래와 같다.

![Euclidean Algorithm](https://i.imgur.com/aa8oGgP.png)

이 알고리즘은 재귀적으로 이용하면 손쉽게 구현할 수 있다. 재귀함수 탈출 조건은 두 수의 모듈로 연산 결과가 0인 경우(두 수 중에서 하나가 1이라면 자연스럽게 모듈로 연산 결과가 0이 된다.)이다. 구현하면 아래와 같다.

```python
def get_gcd(num1, num2):
    return get_gcd(num2, num1 % num2) if num1 % num2 else num2
```

## Irreducible Fraction

위 유클리드 알고리즘을 기약분수 만들기에 적용하면 아래와 같다.

1. **분모, 분자를 대상으로 유클리드 호제법을 통해 최대공약수를 구한다.**
2. **분모, 분자를 최대공약수로 나눈다.**
3. **분모, 분자를 최대공약수로 나누면 두 수는 서로소가 되기 때문에 기약분수가 된다.**

다만 여기서 우리가 입력받는 값은 소수이기 때문에. 분모, 분자의 값을 구하는 코드가 필요하다.

분자의 경우 소수점 이하의 숫자이기 때문에 입력값에 대해 ```split('.')``` 메소드를 이용하면 쉽다. 분모의 값은 분자에 대해서 자릿수를 구하고 그 수만큼 10의 거듭제곱 값을 구하면 된다.

아래와 같이 파이썬으로 구현했다.

## Implementation

```python
# 기약분수 구하기
# 참고 문제 : http://www.codeup.kr/JudgeOnline/problem.php?id=2604


# 유클리드 호제법
def get_gcd(num1, num2):
    return get_gcd(num2, num1 % num2) if num1 % num2 else num2


numerator = input().split('.')[1]
denominator = 10 ** len(numerator)
numerator = int(numerator)

gcd = get_gcd(denominator, numerator)

print('{}/{}'.format(
    numerator//gcd,
    denominator//gcd,
))
```

[CodeUp 2604 : 실수를 기약 분수로 변환]:http://codeup.kr/JudgeOnline/problem.php?id=2604
