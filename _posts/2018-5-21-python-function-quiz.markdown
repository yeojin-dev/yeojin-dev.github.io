---
layout: post
title:  "파이썬 함수 문제풀이"
date:   2018-5-21
categories: [python]
---

<p class="intro"><span class="dropcap">패</span>스트캠퍼스 과제 도전! 함수에 관련된 문제를 풀어보았다.</p>

#### 1. 매개변수로 문자열을 받고, 해당 문자열이 red면 apple을, yellow면 banana를, green이면 melon을, 어떤 경우도 아닐 경우 I don't know를 리턴하는 함수를 정의하고, 사용하여 result변수에 결과를 할당하고 print해본다.

이 문제는 '을 출력할 때 주의해야 한다. 큰따옴표("")를 사용하면 된다.

```python
def color_to_fruit(color):
    if color == 'red':
        return 'apple'
    elif color == 'yellow':
        return 'banana'
    elif color == 'green':
        return 'melon'
    else:
        return "I don't know"
```



#### 2. 1번에서 작성한 함수에 docstring을 작성하여 함수에 대한 설명을 달아보고, help(함수명)으로 해당 설명을 출력해본다.

귀찮아도 docstring을 항상 작성하는 습관을 갖자.

```python
def color_to_fruit(color):
    '''색깔 문자열을 입력하면 대응하는 과일 이름을 출력하는 함수'''
    if color == 'red':
        return 'apple'
    elif color == 'yellow':
        return 'banana'
    elif color == 'green':
        return 'melon'
    else:
        return 'I don\'t know'
    """
    # help(color_to_fruit)
    color_to_fruit(color)
        색깔 문자열을 입력하면 대응하는 과일 이름을 출력하는 함수
    """
```

지금까지 1, 2번은 조건문을 이용해서 작성했지만 딕셔너리를 이용해서 해결할 수도 있다. 아래 코드를 보자.

```python
fruit_dict = {
    'red': 'apple',
    'yellow': 'banana',
    'green': 'melon'
}
def color_to_fruit(color):
    return fruit_dict.get(color, "i don't know")

```

딕셔너리의 get() 메소드는 해당 키의 값을 리턴하지만 2번째 인자로 디폴트값을 설정할 수 있다. 그래서 1줄로 코드를 간결하게 만들 수 있다.



#### 3. 한 개 또는 두 개의 숫자 인자를 전달받아, 하나가 오면 제곱, 두개를 받으면 두 수의 곱을 반환해주는 함수를 정의하고 사용해본다.

*args는 튜플이라는 점이 중요하다.

```python
def calc(*args):
    num = len(args)
    if num == 1:
        return args[0] ** 2
    elif num == 2:
        return args[0] * args[1]
```

위 코드의 경우는 문제점이 하나 존재한다. 바로 인자가 없거나 3개 이상일 경우에 예외처리를 하지 않았다. 그래서 아래와 같이 정리를 해야만 한다. 이것은 선택사항이 아니라 꼭 해야만 한다.

```python
def calc(*args):
    num = len(args)
    if not (num == 1 or num == 2):
      raise ValueError("인자는 1개나 2개만 입력해주세요.")
    if num == 1:
        return args[0] ** 2
    elif num == 2:
        return args[0] * args[1]
```

위치인자 묶음 이외에 함수에 디폴트값을 설정하는 방법도 간결하게 코드를 만들 수 있는 방법이다. 조건표현식으로 더 간결하게 작성할 수도 있다.

```python
def calc(x, y = None):
    return x * (y if y else x)
```



#### 두 개의 숫자를 인자로 받아 합과 차를 튜플을 이용해 동시에 반환하는 함수를 정의하고 사용해본다.

튜플을 이용하면 여러 결괏값을 한번에 리턴해줄 수 있다.

```python
def calc_plus_minus(num1, num2):
    return (num1 + num2, num1 - num2)
```

만약에 차를 절댓값으로 구해야 한다면 abs() 메소드를 사용하면 된다.



#### 4. 위치인자 묶음을 매개변수로 가지며, 위치인자가 몇 개 전달되었는지를 print하고 개수를 리턴해주는 함수를 정의하고 사용해본다.

len() 메소드를 사용하면 간단하다. 중복해서 호출하지 않도록 로컬 변수에 저장하는 코드로 만들었다.

```python
def count_args(*args):
    count = len(args)
    print(count)
    return count
```

#### 5. 람다함수와 리스트 컴프리헨션을 사용해 한 줄로 구구단의 결과를 갖는 리스트를 생성해본다.

람다함수는 이름이 없기 때문에 (람다 표현식)(인자)의 형식으로도 사용할 수 있다.

```python
[ (lambda a, b : f"{a} * {b} = {a * b}")(x, y) for x in range(2, 10) for y in range(1, 10) ]
```

아래와 같이 사용할 수도 있다.

```python
[ f"{x} * {y} = {(lambda x: x * y)(x)}" for x in range(2, 10) for y in range(1, 10) ]
```
