---
layout: post
title:  "Model field options"
date:   2018-6-21
categories: [django]
---

<p class="intro"><span class="dropcap">D</span>jango의 models 패키지에는 웹 애플리케이션의 데이터 모델링 정보를 담을 수 있는 Model 클래스가 있다. 이 클래스의 인스턴스 변수를 통해 데이터베이스의 필드를 설정할 수 있다.</p>

인스턴스 변수는 Field 클래스를 상속받은 클래스의 인스턴스를 가리키며 이는 데이터베이스의 필드가 된다. 데이터베이스의 필드는 여러 가지 추가 속성을 갖는데 이것은 각 클래스의 인자를 통해서 구현할 수 있다. 이것을 Django에서 Field Options로 설명하고 있다.

원문은 여기를 참고하자. Django 버전은 2.0이 기준이다. (이 포스트는 번역 포스트가 아니다. 하지만 포스트의 내용이 많이 들어가 있으며 보다 쉽게 설명하는 식으로 쓰려고 노력했다.)

**[Model field reference 원문(Django Document)]**

## Field Options

#### Field.null

만약 True라면 Django는 빈 값으로 Null을 입력한다. 기본 설정은 False이다. Django 공식 문서에서는 문자열 필드(CharField, TextField)에 대해 이 옵션을 False로 하기를 권장한다. 왜냐하면 Null을 인정하면 '빈 문자열'에 대해 2개의 값이 가능하게 되기 때문이다. Django는 Null보다 빈 문자열을 사용하는 것을 규칙으로 하고 있다.

참고로 BooleanField에서 Null을 허용하고 싶다면 NullBooleanField를 사용해야 한다.

#### Field.blank

만약 True라면, 필드는 빈 값을 허용한다. 기본값은 False이다. Field.blank와 Field.null은 같지 않다. 전자는 유효성 검사를 위해 사용하는 옵션이며, 후자는 데이터베이스를 위한 옵션이다. 만약 blank=True라면 빈 값 역시 유효성 검사를 통과한다.

#### Field.choices

마치 다른 HTML의 \<select\>와 같이 2중 튜플 구조를 사용해서 셀렉트 박스를 사용할 수 있게 한다. 이 옵션은 약어를 사용하고 싶을 때 큰 도움이 된다.

아래 코드를 보자.

```python
from django.db import models

class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = (
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    )
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)
```

여기서 학년을 'FR', 'SO' 등의 약어로 표현할 수 있도록 Model 클래스를 설계했음을 알 수 있다. 가독성에서 매우 뛰어나다는 것을 알 수 있는데 아래 is_upperclass() 메소드의 구현을 보면 어떻게 사용해야 할지 감이 올 것이다.

만약 여기서 약어의 원래 단어('Freshman')을 사용하고 싶다면 get_FOO_display()라는 형식의 메소드로 사용할 수 있다.

조금 더 고급 사용법을 설명하자면 2중 튜플 구조를 응용해서 단순한 약어 처리가 아닌 더 복잡한 정보를 표현할 수 있다. 예시는 아래와 같다.

```python
MEDIA_CHOICES = (
    ('Audio', (
            ('vinyl', 'Vinyl'),
            ('cd', 'CD'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
)
```

#### Field.db_column

이 옵션을 통해 데이터베이스의 컬럼명을 별도로 설정할 수 있다. 이 옵션이 없다면 Django는 자동적으로 필드명을 컬럼명으로 사용한다. 특정 문자를 컬럼명으로 사용할 수 없는 데이터베이스에서 효과적이다.

#### Field.index

만약 True라면 데이터베이스 인덱스는 이 옵션을 가진 필드에 생성된다.

#### Field.db_tablespace

이 필드가 인덱싱되어 있다면 설정한 테이블스페이스의 인덱스로서 이 필드를 사용할 수 있다. 테이블스페이스란 데이터베이스를 저장하는 물리적인 공간을 말한다.

#### Field.default

이 필드의 기본값을 설정한다.

#### Field.editable

만약 False의 값을 갖고 있다면 admin 및 다른 ModelForm에서 나타나지 않는다. 유효성 검사 역시 적용되지 않는다. 기본값은 True이다.

#### Field.error_messages

이름에서도 알 수 있듯이 여러 상황에서 사용하는 에러 메시지를 오버라이드한다. 이 옵션은 딕셔너리를 인자로 넣어줘야 하는데 필요한 key는 아래와 같다. 이름을 보면 알겠지만 해당 키가 가리키는 옵션에서 예외가 발생했을 때 메시지를 value로 설정해줄 수 있다.

- null
- black
- invaild
- invalid_choice
- unique
- unique_for_date

#### Field.help_text

form 위젯에 나올 추가 도움 텍스트를 추가할 수 있다. 이 옵션의 텍스트는 이스케이프 처리가 되지 않기 때문에 HTML 태그를 사용할 수 있다.

#### Field.primary_key

True 값이라면 필드가 해당 테이블의 기본키가 된다. 기본키가 되기 때문에 이 필드는 자연스럽게 null=False, unique=True가 된다.

False 값이라면 Django 프레임워크에서 자동적으로 기본키 필드(id)를 만들어준다.

#### Field.unique

True 값이라면 해당 필드는 고유한 값을 가져야 한다. 만약 중복된 값이 있다면 save() 메소드에서 IntegrityError 예외가 발생한다.

이 옵션은 ManyToManyField, OneToOneField를 제외한 모든 필드에 적용가능하며, 필드의 모든 값이 고유한 값이라면 그 필드는 인덱스가 될 수 있기 때문에 unique=True 옵션이 있다면 db_index는 굳이 추가하지 않아도 된다.

#### Field.unique_for_date

이 옵션에 DateField, DateTimeField 필드를 설정해줌으로써 이 옵션을 가진 필드가 어떤 날짜에 대해 유니크함을 알려준다.

예를 들어, title이라는 필드에 unique_for_date='pub_date'라는 옵션이 설정되어 있다면, 특정 pub_date(이 필드는 DataField이거나 DateTimeField이거나 둘 중에 하나일 것이다.)에 대해 하나의 title만 존재할 것이다.

이 옵션은 Model.validate_unique()라는 메소드에 의해 유효성 검사를 하지만 데이터베이스 단계에서 처리하는 것은 아니다.

#### Field.unique_for_month

unique_for_date와 같지만 기준이 월(month)이다.

#### Field.unique_for_year

unique_for_date와 같지만 기준이 연도(year)이다.

#### Field.verbose_name

필드명에 대해서 가독성이 높은 이름(A human-readable name)를 지원해준다. 설정해주지 않으면 변수명이 그대로 적용된다.

#### Field.validators

해당 필드에서 사용할 유효성 검사 validator[^1]을 설정해준다. 이 validator는 필드 클래스 내에서 유효성 검사를 해준다.



[^1]: validator는 데이터를 인자로 받아 조건에 따라 ValidationError 예외를 발생시키는 콜러블이다. Django에서는 validator를 만들어주는 클래스들이 있는데 RegexValidator 등이 있다.

[Model field reference 원문(Django Document)]: https://docs.djangoproject.com/en/2.0/ref/models/fields/
