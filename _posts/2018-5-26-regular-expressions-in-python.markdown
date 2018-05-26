---
layout: post
title:  "Regular Expressions in Python"
date:   2018-5-26
categories: [python]
---

<p class="intro"><span class="dropcap">R</span>Regular Expressions(정규표현식)은 문자열을 다루는 데 매우 유용한 표현식이다. 정규표현식을 사용해서 문자열을 원하는 대로 검색할 수도 있고 변환시킬 수도 있다.</p>

정규표현식의 기본 문법은 아래와 같다.

## 패턴 문자

패턴|문자
---|---
\\d|숫자
\\D|비숫자
\\w|문자
\\W|비문자
\\s|공백 문자
\\S|비공백 문자
\\b|단어 경계 (\w와 \W의 경계)
\\B|비단어 경계

## 패턴 지정자(Pattern specifier)

패턴|의미
---|---
abc|리터럴 `abc`
(expr)|expr
expr1 \| expr2 | expr1 또는 expr2
`.` | `\n`을 제외한 모든 문자
`^` | 소스문자열의 시작
`$` | 소스문자열의 끝
expr`?` | 0 또는 1회의 expr
expr`*` | 0회 이상의 최대 expr
expr`*?`| 0회 이상의 최소 expr
expr`+` | 1회 이상의 최대 expr
expr`+?`| 1회 이상의 최소 expr
expr`{m}`| m회의 expr
expr`{m,n}`| m에서 n회의 최대 expr
expr`{m,n}?` | m에서 n회의 최소 expr
[abc] | a or b or c
[^abc] | not (a or b or c)
expr1(?=expr2) | 뒤에 expr2가 오면 expr1에 해당하는 부분
expr1(?!expr2) | 뒤에 expr2가 오지 않으면 expr1에 해당하는 부분
(?<=expr1)expr2 | 앞에 expr1이 오면 expr2에 해당하는 부분
(?<!expr1)expr2 | 앞에 expr1이 오지 않으면 expr2에 해당하는 부분

## 파이썬에서 정규표현식 사용하기

파이썬에서 정규표현식을 쓰기 위해서는 <re> 패키지를 import해야만 한다. re 패키지가 제공하는 주요 메소드는 아래와 같다.

- **match** : 시작부터 일치하는 패턴 찾기

match() 메소드는 해당 정규표현식과 대상이 되는 문자열의 매치 정보를 담은 match 타입의 오브젝트를 반환한다. 여기서 일치하는 패턴의 데이터를 얻고 싶으면 match 인스턴스의 groups() 메소드를 사용하면 된다.

그리고 이 메소드는 시작부터 일지하는 패턴을 찾는다. 처음부터 패턴과 일치하지 않으면 None을 리턴한다.

```python
import re
m = re.match(expr, source)
```

- **search** : 첫 번째 일치하는 패턴 찾기

search()는 match()와 달리 문자열 전체에서 일치하는 부분을 찾다. 리턴 타입은 match()와 같다.

```python
import re
m = re.search(expr, source)
```

- **findall** : 일치하는 모든 패턴 찾기

아마 정규표현식에서 가장 유용한 메소드는 findall() 메소가 아닌가 싶다. 이름 그대로 이 메소드는 패턴과 일치하는 문자열을 리스트로 리턴해준다.

```python
import re
m = re.findall(expr, source)
```

- **split** : 패턴으로 나누기

문자열(str 타입)의 split() 메소드와 비슷하다. 정규표현식에서 사용하면 패턴을 기준으로 나눌 수 있다.

```python
import re
m = re.split(expr, source)
```

- **sub** : 패턴 대체하기

문자열의 replace() 메소드처럼 패턴에 일치하는 문자열을 바꿔준다.

```python
import re
m = re.sub(expr, repl, source)
```

sub() 메소드의 경우는 repl에 메소드를 인자로 사용할 수 있다. 아래 코드를 보자.

```python
def dashrepl(matchobj):
    if matchobj.group(0) == '-': return ' '
    else: return '-'
re.sub('-{1,2}', dashrepl, 'pro----gram-files')
```

위 코드는 하이픈이 1개일 경우 빈 칸으로, 2개일 경우 하이픈 하나로 줄여주는 코드이다. 인자로 넘겨준 'pro----gram-files' 문자열은 'pro--gram files'으로 바뀌게 된다. 그리고 메소드를 인자로 받으니 람다함수도 당연히 가능하다. 위 코드를 아래와 같이 바꿔서 쓸 수 있다.

```python
re.sub('-{1,2}', (lambda x: ' ' if x == '-' else '-'), 'pro----gram-files')
```

다만 sub() 메소드에서 람다함수를 사용하는 것은 가독성을 해칠 우려가 있는 것 같다. 안그래도 복잡한 암호같은 정규표현식에 람다함수까지 사용한다는 것은 조금 무리일 수도 있다.