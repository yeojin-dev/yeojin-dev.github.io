---
layout: post
title:  "Django Models"
date:   2018-6-18
categories: [django]
---

<p class="intro"><span class="dropcap">D</span>jango models 패키지는 각 테이블의 스키마를 설정하고 테이블 사이의 관계를 나타내준다.</p>

Django는 ORM 기법을 사용한다. ORM 기법이란 DB의 테이블을 하나의 클래스로 관리하는 기법인데 Django의 models 패키지는 이 ORM 기법을 구현하는 클래스이다.

DB를 공부하면 알 수 있듯이, 각각의 테이블은 서로 독립적일 수도 있고 서로 연관이 있을 수도 있다. (DB 설계에서 이는 '정규화'라고 부른다.) 테이블 사이의 연관성은 크게 OTM(One To Many), MTM(Many To Many), OTO(One To One)으로 나눌 수 있는데 Django에서는 ORM 기법을 사용하기 때문에 models 패키지의 각 필드 타입으로 위 3가지 관계를 구현하고 있다.

## OTM - ForeignKey

OTM 관계는 다대일 관계로, 두 테이블에서 하나의 레코드와 다른 테이블의 여러 레코드가 관계를 가질 경우에 사용한다. 실생활에서는 블로그 글(하나)에 달리는 여러 댓글을 생각해볼 수 있다.

Django에서는 이 경우에 ForeignKey 필드를 구현함으로써 구현할 수 있다.

```python
from django.db import models


class Car(models.Model):
    manufacturer = models.ForeignKey(
        'Manufacturer',
        on_delete=models.CASCADE,
    )
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name


class Manufacturer(models.Model):
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name
```

위 코드를 보면 하나의 Manufacturer에 대해 여러 Car 레코드가 존재할 수 있도록 설계되어 있다. 하나의 자동차 메이커가 여러 종류의 자동차를 만들 수 있는 것과 같은 맥락이다.

## MTM - ManyToManyField

MTM 관계는 다대다 관계로, 두 테이블의 레코드 여럿이 서로 연관이 있을 때의 관계이다. 실생활에서는 친구관계를 생각해볼 수 있다. 한 사람은 여러 사람을 친구로 둘 수 있고, 그것은 다른 사람들도 마찬가지이다.

Django에서는 이 경우에 ManyToManyField 필드를 구현함으로써 구현할 수 있다.

```python
from django.db import models


class Topping(models.Model):
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name


class Pizza(models.Model):
    name = models.CharField(max_length=50)
    toppings = models.ManyToManyField(
        Topping,
        related_name='pizzas',
    )

    def __str__(self):
        return self.name
```

피자와 토핑은 서로 다대다 관계이다. 하나의 피자는 여러 토핑을 가지고 있고, 토핑은 여러 피자의 재료가 된다. 이럴 때 Pizza 클래스에서 Topping을 ManyToManyField 필드(toppings)로 설정해놓으면 가능하다.

참고로, ManyToManyField 필드를 설정하면 Django는 별도의 테이블을 따로 생성하고 두 테이블 사이의 관계를 나타내는 Record를 저장해둔다.

## OTO - OneToOneField

테이블 사이에 1:1 관계를 설정할 때는 OneToOneField 필드를 사용하면 된다. OTO 관계는 객체지향 프로그래밍에서 상속 관계와 매우 유사하다. (실제로 상속으로도 구현이 가능하다.)

OTO 관계를 파이썬 코드로 나타내면 아래와 같다.

```python
from django.db import models


class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

    def __str__(self):
        return "%s the place" % self.name

class Restaurant(models.Model):
    place = models.OneToOneField(
        Place,
        on_delete=models.CASCADE,
        primary_key=True,
    )
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

    def __str__(self):
        return "%s the restaurant" % self.place.name
```

Place는 각각의 장소를 나타내는 테이블이고 Restaurant 테이블은 음식점을 나타낸다. 하나의 음식점은 하나의 장소에만 존재할 수 있으며, 하나의 장소에는 하나의 음식점만 들어올 수 있다. 이런 관계를 OneToOneField를 사용해 구현할 수 있다.
