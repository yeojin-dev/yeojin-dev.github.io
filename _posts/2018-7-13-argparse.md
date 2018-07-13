---
layout: post
title:  "argparse 라이브러리로 커맨드라인 인자 활용하기"
date:   2018-7-13
categories: [python]
---

<p class="intro"><span class="dropcap">a</span>rgparse 라이브러리를 활용하면 .py 파일을 실행시킬 때 적절한 인자들을 입력받을 수 있다. 이는 Django 배포 자동화 등등 다양한 분야에서 활용할 수 있다.</p>

파이썬 문서에서 [argparse] 부분에 나온 예제 코드를 확인해보자.

```python
import argparse

parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                    help='an integer for the accumulator')
parser.add_argument('--sum', dest='accumulate', action='store_const',
                    const=sum, default=max,
                    help='sum the integers (default: find the max)')

args = parser.parse_args()
print(args.accumulate(args.integers))
```

위 코드를 뜯어보면서 argparse 라이브러리 사용법을 정리해보았다.

## argparse.ArgumentParser()

```python
parser = argparse.ArgumentParser(description='Process some integers.')
```

argparse.ArgumentParser 클래스의 인스턴스 생성은 argparse 라이브러리를 활용하기 위한 첫 번째 단계이다. 이 인스턴스는 커맨드라인 인자들을 파이썬 객체로 저장하기 위한 정보를 담고 있다.

## argparse.ArgumentParser.add_argument()

add_argument() 메소드는 커맨드라인 인자 전반을 관리하게 해주는 메소드이다. 사실상 argparse 라이브러리의 핵심 메소드라고 볼 수 있다. 이 메소드의 패러미터를 정리하면 아래와 같다.

```
ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest])¶
```

1. name or flags - 입력받을 인자를 나타내는 이름이다.
2. action - add_argument() 메소드에서 설정한 인자를 입력받았을 때 하는 행동, store, append, count 등이 있다.
3. nargs - 해당 인자를 몇 개나 받을 수 있는지 설정으로 숫자, ?, \*, +등이 올 수 있다.
4. const - action='store_const'나 nargs='?'의 기본값으로 설정하는 값. 콜러블 설정도 가능하다.
5. default - 인자가 생략되었을 때에는 None을 기본값으로 입력받으나 이 패러미터 설정으로 기본값을 바꿀 수 있다. default 역시 콜러블 설치가 가능하다.
6. type - 인자로 입력하는 값의 데이터타입을 정할 수 있다. int, float 등이 대표적이다. 하지만 콜러블을 설정함으로써 일종의 유효성 검사 처리도 가능하다.
7. choices - 열거형을 설정해서 특정 문자열의 인자만 받을 수 있다.
8. required - 생략할 수 있는지 없는지 설정할 수 있다.
9. help - 인자에 대한 간단한 설명글이다.
10. metavar - 인자의 이름인데 코드 내 이름이 아닌 사용자가 알기 쉽게 새로 쓴 이름이다. 예를 들어 코드 내 변수가 'num'이지만 metavar는 'number'로 설정할 수 있다.
11. dest - parse_args() 메소드에서 입력받은 인자들을 네임스페이스 형태[^1]로 리턴받을 수 있다. 이 때 dest 패러미터를 설정하면 딕셔너리에 어떤 이름의 key를 설정할 수 있고 이 key에 value로서 인자를 입력받아 리턴받을 수 있다.

#### 복수의 인자 리스트로 처리하기

```python
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                    help='an integer for the accumulator')
```

이 메소드는 1개 이상의 정수 타입 인자를 integers라는 이름의 리스트로 관리하게 해준다.

```python
parser.add_argument('--sum', dest='accumulate', action='store_const',
                    const=sum, default=max,
                    help='sum the integers (default: find the max)')
```

이 메소드는 `--sum`이라는 이름의 인자를 입력하면 네임스페이스에 sum() 빌트인 함수가, 입력이 없으면 max() 빌트인 함수가 네임스페이스에 저장된다. 그리고 네임스페이스에는 accumulate라는 속성의 값으로 이 함수가 저장될 것이다.

```python
args = parser.parse_args()
print(args.accumulate(args.integers))
```

위 코드를 보면 parser.parse_args() 메소드로 네임스페이스 args를 리턴받았다. 그리고 args 네임스페이스의 accumulate 속성에 저장된 함수로 integers에 저장된 정수들을 계산하고 있음을 알 수 있다.

[^1]:argparse 라이브러리에서 사용하는 클래스이다. 딕셔너리와 매우 비슷하며, var() 빌트인 함수의 인자로 네임스페이스 인스턴스를 넣으면 딕셔너리를 얻을 수 있다.
[argparse]:https://docs.python.org/3/library/argparse.html
