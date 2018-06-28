---
layout: post
title:  "Test Code in Django"
date:   2018-6-28
categories: [django]
---

<p class="intro"><span class="dropcap">T</span>DD(Test-Driven Development)는 소프트웨어 개발 방법론 중 하나이다. 기존의 개발 방법이 설계-개발-테스트-평가-개발로 이어지는 것에 비해 TDD는 설계 단계에서 코드로 정확하게 목적과 목표를 정의하고 이를 확인할 수 있는 테스트 코드를 작성한다.</p>

TDD를 파이썬 기본 라이브러리에서도 지원하지만 Django 역시 TDD를 위한 패키지를 지원한다. 그렇다면 Django 프로젝트 내에서는 어떻게 코드를 작성해야 할까?

## unittest.TestCase

먼저 파이썬 기본 라이브러리를 보자. 간단한 수준에서는 파이썬에서 지원하는 TestCase 클래스의 메소드를 이용하면 된다. 메소드를 정리하면 아래와 같은데 여기서 말하는 '의미'란 해당 조건을 통과해야 한다는 것을 의미한다.

| 메소드 | 의미 |
|:---------|:-----------------|
| assertEqual(a, b) | a == b |
| assertNotEqual(a, b) | a != b |
| assertTrue(x) | bool(x) is True |
| assertFalse(x) | bool(x) is False |
| assertIs(a, b) | a is b |
| assertIsNot(a, b) | a is not b |
| assertIsNone(x) | x is None |
| assertIsNotNone(x) | x is not None |
| assertIn(a, b) | a in b |
| assertNotIn(a, b) | a not in b |
| assertIsInstance(a, b) | isinstance(a, b) |
| assertNotIsInstance(a, b) | not isinstance(a, b) |

## TestCase in django

그렇다면 위 메소드들을 Django에서 어떻게 사용하면 좋을까? Django의 test 패키지를 이용하면 된다.

아래 예제 코드를 보자.

```python
from django.contrib.auth import get_user_model
from django.test import TransactionTestCase


User = get_user_model()


class RelationTestCase(TransactionTestCase):
    def test_following(self):
        u1 = User.objects.create_user(username='u1')
        u2 = User.objects.create_user(username='u2')

        self.assertTrue(u1.following_relations.filter(to_user=u2).exists())
```

위 코드를 보면 User 클래스의 인스턴스 u1이 'following_relations'라는 매니저(related_name)에 대해 'to_user=u2'의 조건에 해당하는 쿼리셋이 존재하는지 확인하는 테스트 코드임을 알 수 있다. assertTrue() 메소드의 특성상 이 쿼리셋의 결과가 적어도 하나 이상 존재함을 개발자는 가정하고 있다.

테스트 코드를 실행시키기 위해서는 아래와 같이 실행하면 된다.

```
python manage.py test
```

위 명령어의 실행 결과를 통해 현재 Django 프로젝트에서 u1.following_relations.filter(to_user=u2).exists()의 결과가 True인지 False인지 콘솔에서 알 수 있다. 만약 다른 코드에 문제가 생겨서 이 값이 False가 되는 상태라면 이 테스트 코드를 통해 어딘가 문제가 생겼음을 알 수 있는 것이다.

불편하게 직접 쉘에서 작업하는 것보다 테스트 코드를 사용하면 테스트 내용을 공유할 수도 있고 훨씬 편리하게 작업할 수 있다.
