---
layout: post
title:  "Decorator"
date:   2018-5-31
categories: [python]
---

<p class="intro"><span class="dropcap">D</span>ecorator는 파이썬에서 다른 메소드를 콜러블(callable)로 변환시켜주는 메소드를 말한다. 데커레이터를 사용하면 중복되는 코드를 줄일 수 있어서 코드 생산성이 높아진다.</p>

## Basic

아래와 같은 2개의 메소드가 있다고 가정해보자.

```python
def func1():
  print("Hello!")

def func2():
  print("World!")
```

만약 이 2개의 메소드에 "=== Log ==="라는 새로운 문자열 출력을 추가하고 싶다고 한다면 어떻게 해야할까? 아마도 이렇게 해야할 것이다.

```python
def func1():
  print("=== Log ===")  # 추가 문자열
  print("Hello!")

def func2():
  print("=== Log ===")  # 추가 문자열
  print("World!")
```

메소드가 2개만 있다면 그냥 넣는 것도 나쁘지 않다. 그런데 이런 메소드가 만약 10개라면? 아니 100개라면? 그리고 단순한 문자열 출력이 아니라 어려운 알고리즘의 구현이라면? 개발자로서 가장 하기 싫은(그리고 위험할 수도 있는) 반복작업을 해야할지도 모른다. 이럴 때 사용하는 것이 데커레이터이다.

위 코드를 아래와 같이 바꿔보자.

```python
def func1():
  print("Hello!")

def func2():
  print("World!")

def print_log(func):
    def inner_func():
        print("===Log===")
        result = func
        return result()
    return inner_func

new_func1 = print_log(func1)
new_func2 = print_log(func2)

new_func1()
# ===Log===
# Hello!

new_func2()
# ===Log===
# World!
```

코드를 살펴보면 print_log()라는 메소드가 func1(), func2() 메소드에게 inner_func() 메소드 내용을 장식해주는 것 같은 모양새이다. 여기서 print_log() 메소드를 데커레이터라고 부른다.

데커레이터는 @(데커레이터 이름)의 형태로 보다 간략하게 만들 수 있다.

```python
def print_log(func):
    def inner_func():
        print("===Log===")
        result = func
        return result()
    return inner_func

@print_log  # 데커레이터
def func1():
    print("Hello!")

@print_log  # 데커레이터
def func2():
    print("World!")

func1()
# ===Log===
# Hello!

new_func2()
# ===Log===
# World!
```

특이한 점은 print_log라는 이름의 데커레이터로 인해 **func1() 메소드는 func1() 자신이 아닌 실제로는 inner_func()를 실행한다는 점**이다. 이렇게 func1() 메소드가 inner_func() 메소드를 부르는, 메소드가 다른 메소드를 부르는 타입의 메소드를 **'콜러블'**이라고 부른다. 즉, 데커레이터는 콜러블을 만들어주는 역할을 하는 메소드이다.

하지만 데커레이터는 보통의 콜러블과 달리 **함수 사이의 관계를 보다 잘 나타낼 수 있다.** 이른바 메타프로그래밍이 가능하다는 것이다. 위 파이썬 코드에서도 데커레이터로 인해 print_log() 메소드와 func1(), func2() 메소드 사이의 종속관계를 직관적으로 알 수 있다.

## import time

데커레이터의 또다른 특징은 **데커레이터의 코드는 정의된 직후에 실행된다는 점**이다. 이 시점은 모듈이 임포트되는 시점(import time)과 일치한다. 다시 print_log()를 보자.

```python
def print_log(func):
    print("START!")  # import time에 실행됨
    def inner_func():
        print("===Log===")
        result = func
        return result()
    return inner_func
```

내부 메소드 inner_func() 부분이 아닌 print_log()의 코드들은 import time에 실행된다. 즉, 데커레이터는 일종의 초기화 기능을 담당하고 있다. 그래서인지 일반적으로는 데커레이터와 콜러블을 분리해서 구현한다고 한다.
