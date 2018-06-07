---
layout: post
title:  "French Deck in Python (1)"
date:   2018-6-7
categories: [python]
---

<p class="intro"><span class="dropcap">P</span>ython으로 트럼프 카드를 구현해보자. 파이썬 데이터 모델 중 Collections.namedtuple 클래스가 도움이 될 것이다.</p>

## Basic

FrenchDeck이라는 이름의 클래스를 만들고 Card라는 인스턴스들을 항목으로 가진 리스트(Cards)를 FrenchDeck 클래스의 속성으로 구현해보자. 여기서 Card를 구현할 때 기본 자료형을 쓴다면 어떻게 될까? 나쁘지는 않지만 나라면 리스트와 튜플을 이용하려고 할 것이다. [('clubs', '7'),] 이런 식으로 말이다. 하지만 이 구현은 단점을 가지고 있다. 예를 들면 해당 튜플과 튜플의 각 항목이 무엇을 의미하는지 나타내고 있지 않고 있다. 이에 반해 namedtuple은 튜플의 항목을 마치 DB의 속성처럼 표현할 수 있다.

그리고 FrenchDeck 클래스가 가져야 하는 메소드를 정의해보면 아래와 같다.

| 메소드 이름 | 기능 |
|:--------:|:----------------:|
| length | 트럼프 카드의 총 카드 수 |
| get | 해당 순서의 트럼프 카드 리턴 |

파이썬에서는 위의 두 메소드는 직접 구현하기보다 매직 메소드를 이용하는 것이 더 효과적이다. length의 경우 \_\_len\_\_(), get의 경우 \_\_getitem\_\_()을 구현하면 된다.

그럼 이제 아래 코드를 보자.

## Implementation

```python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
  ranks = [str(n) for n in range(2, 11)] + list('JQKA')
  suits = ['clubs', 'hearts', 'diamonds', 'spades']

  def __init__(self):
    self._cards = [Card(rank, suit) for suit in self.suits
                                    for rank in self.ranks]

  def __len__(self):
    return len(self._cards)

  def __getitem__(self, position):
    return self._cards[position]


# test code
deck = FrenchDeck()
len(deck)  # 52
deck[0]  # Card(rank='2', suit='clubs')
deck[-1]  # Card(rank='A', suit='spades')
```

상당히 직관적이고 간결하게 트럼프 카드를 표현할 수 있게 되었다. 그렇다면 트럼프 카드 중에서 무작위로 카드를 뽑을 때는 어떻게 하면 좋을까? 아래와 같이 하면 된다.

```python
from random import choice:
  choice(deck)  # 랜덤한 Card 리턴
```

생각보다 아주 짧게 가능하다는 것을 알 수 있다. 이처럼 파이썬의 훌륭한 장점 중 하나는 파이썬의 표준 라이브러리와 매직 메소드를 통해 일관적인 코딩이 가능하다는 점이다.

## 파이썬 데이터 모델의 장점

1. 사용자의 일반적인 연산을 위해 각 클래스가 구현한 메소드를 암기할 필요가 없다.
2. 표준 라이브러리의 기능을 바로 클래스에 사용할 수 있다.

이 외에도 여러 매직 메소드를 구현할 수 있다. 적절한 파이썬 데이터 모델을 사용하는 것이 훌륭한 파이썬 개발자가 되는 지름길이다.
