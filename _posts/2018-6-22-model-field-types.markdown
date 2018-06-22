---
layout: post
title:  "Model field types"
date:   2018-6-22
categories: [django]
---

<p class="intro"><span class="dropcap">D</span>jango의 models 패키지의 Model 클래스는 데이터베이스의 필드가 되는 여러 클래스들을 설정하고 있다. 이 클래스는 필드의 성격에 따라 구분되어 있다.</p>

데이터베이스에서 필드의 자료형을 number, date, varchar 등으로 나누듯이 Django에서는 클래스로 이를 구분[^1]한다. 이 포스트에서는 Django의 Model에는 어떤 클래스들이 있는지(어떤 필드가 있는지) 살펴볼 것이다.

원문은 여기를 참고하자. Django 버전은 2.0이 기준이다. (이 포스트는 번역 포스트가 아니다. 하지만 포스트의 내용이 많이 들어가 있으며 보다 쉽게 설명하는 식으로 쓰려고 노력했다.)

**[Model field reference 원문(Django Document)]**

## Field Options

#### AutoField

AutoField는 ID와 같이 자동적으로 증가하는 IntegerField를 말한다. Django에서는 자동적으로 ID 필드를 만들어주기 때문에 반드시 AutoField가 필요하지는 않다.

#### BigAutoField

AutoField와 같지만 BigAutoField는 1에서 9223372036854775807까지의 범위를 갖는다.

#### BigIntegerField

BigIntegerField는 64비트 정수 자료형이다. 이 필드는 -9223372036854775808에서 9223372036854775807까지의 값을 갖는다. 특이하게도 이 필드의 Widget은 TextInput가 기본값이다.

#### BinaryField

이진 데이터를 가질 수 있는 필드이다. 이진 데이터만 가능하기 때문에 바이트 단위로 할당(assign)할 수 있다. 센서 데이터 등 이진 데이터가 필요한 경우에 사용하면 좋을 것 같다.

BinaryField는 filter 등의 QuerySet 관련 메소드를 사용할 수 없다.

#### BooleanField

True/False 값만 갖는 필드이다. Boolean 타입의 값을 갖기 때문에 기본 widget의 값이 CheckboxInput이다. Null 값이 필요한 필드라면 NullBooleanField를 사용해야만 한다.

#### CharField

문자열 값을 갖는 필드이다. 아주 많은 양의 문자열이 필요하다면 TextField를 사용하는 것이 좋다. TextInput이 기본 widget이다. max_length 옵션을 통해 길이 제한을 둘 수 있다.

#### DateField

파이썬의 datetime.date 인스턴스를 통해 날짜를 입력할 수 있다. 이 필드에는 유용한 인자가 있다.

- **DataField.auto_now** : 자동으로 현재 날짜를 입력해준다. 이 인자는 Model.save() 메소드를 호출하는 순간에 생성된다. 그러므로 이것은 업데이트 될 때마다 갱신된다. '가장 최근 수정' 날짜를 입력할 때 효과적이다.

- **DataField.auto_now_add** : 자동으로 현재 날짜를 입력해준다. 이 인자는 인스턴스가 생성될 때 입력된다. 즉, 처음 1번만 생성된다. 이것은 '생성된 날짜'를 입력할 때 효과적이다. 주의할 점은 생성될 때 입력되는 값이기 때문에 인스턴스 생성 시에 어떤 값을 입력되어도 그 값은 무시된다.

만약 어떤 인스턴스를 만들 때 auto_now_add 대신에 임의로 입력한 값을 사용하고 싶다면 auto_now_add=True 대신에 default 인자를 설정해야 한다.

#### DateTimeField

DateField와 같지만 datetime.datetime 인스턴스를 사용한다.

#### DecimalField

고정소수점형 10진수 데이터타입인 Decimal을 지원하는 필드이다. Decimal은 정해진 precision 내에서 엄격한 연산을 해야하는 경우에 사용한다.

#### DurationField

어느 기간 동안의 시간을 나타내는 필드이다. 파이썬의 timedelta 인스턴스를 사용한다. 데이터베이스 차원에서 보자면 PostgreSQL에서는 interval 데이터타입을 사용하며 오라클에서는 INTERVAL TO를 사용한다. 다른 경우에는 밀리초를 저장하는 bigint 타입이 된다.

#### EmailField

CharField와 같지만 메일 주소 유효성 검사를 지원한다. Django의 내장 validator인 EmailValidator를 사용한다.

#### FileField

파일을 업로드할 때 사용하는 필드이다. 이 필드는 기본키가 될 수 없다.

- **FileField.upload_to** : 파일이 저장될 위치를 설정한다. settings.py의 MEDIA_ROOT에 설정된 위치의 하위 위치를 지정할 수 있다. 지정될 위치를 직접 지정할 수도 있지만 콜러블을 사용할 수도 있다. 아래 코드를 보자. user_directory_path()의 리턴값을 저장위치로 사용함을 알 수 있다. (instance는 FileField가 설정된 인스턴스이며 filename은 업로드한 파일의 이름을 말한다.)

```python
def user_directory_path(instance, filename):
    # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
    return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
    upload = models.FileField(upload_to=user_directory_path)
```

- **FileField.storage** : 파일을 저장, 관리하는 FileSystemStorage 인스턴스를 지정한다. 업로드한 파일을 덮어쓰기해야하거나 이름을 바꿔야 하는 경우에 해당 기능을 담당하는 FileSystemStorage 인스턴스를 생성해서 지정해주면 된다.

덧붙이자면 이 필드를 사용할 때 FieldFile 클래스를 사용해 파일 자체를 관리한다.

#### FilePathField

CharField와 같지만 특정 디렉토리에 있는 파일이름을 지정하는데 사용된다. 이 필드는 5개의 인자가 있는데 이 중에서 1번째는 필수이다. 이 필드는 기본적으로 max_length가 100으로 설정되어 있다.

- **FilePathField.path** : FilePathField가 가질 수 있는 파일이름들이 있는 디렉토리
- **FilePathField.match** : 특정 요건에 맞는 파일이름을 갖도록 필터링해주는 정규표현식
- **FilePathField.recursive** : True일 경우 하위 디렉토리의 파일이름도 필드값이 될 수 있다. 기본값은 False이다.
- **FilePathField.allow_files** : 지정한 위치의 파일을 포함할지를 True/False로 설정한다. 기본값은 True이다.
- **FilePathField.allow_folders** : 지정한 위치의 디렉토리를 포함할지를 True/False로 설정한다. 기본값은 False이다.

#### FloatField

파이썬의 float 타입을 저장하는 필드이다.

#### ImageField

FileField를 상속받은 클래스로서 이미지 파일을 업로드하도록 유효성 검사를 지원한다. 이미지 파일을 위한 필드이기 때문에 다음 2개의 인자를 입력받을 수 있다.

- **ImageField.height_field** : 이미지의 세로 크기를 의미하는 필드를 생성하고 싶을 때 사용한다. 이 인자를 지정해주면 해당 이름의 필드가 생성되고 이미지의 세로 크기를 저장해준다.
- **ImageField.width_field** : 위 인자와 같은데 이것은 이미지의 가로 크기를 의미한다.

#### IntegerField

정수형 데이터를 필드값으로 갖는 필드. 32비트 정수형 데이터이기 때문에 값의 범위는 -2147483648에서 2147483647이다. NumberInput을 기본 widget으로 갖는다.

#### GenericIPAddressField

IPv4 또는 IPv6 IP 주소를 필드값으로 갖는 필드이다. 아래의 2개의 인자를 입력받을 수 있다.

- **GenericIPAddressField.protocol** : 어떤 프로토콜을 사용할지 결정한다. 'both', 'IPv4', 'IPv6'가 가능하다.
- **GenericIPAddressField.unpack_ipv4** : IPv6에 IPv4 주소를 대응시킬 수 있는데 이 경우 IPv4를 언패킹[^2]하기 위해 이 패러미터를 사용한다.

#### NullBooleanField

BooleanField와 같지만 Null을 하나의 옵션으로 사용할 수 있는 필드이다.

#### PositiveIntegerField

IntegerField와 같지만 0과 양수만 가능하다. 범위는 0에서 2147483647이다.

#### PositiveSmallIntegerField

PositiveIntegerField와 같지만 범위가 0에서 32767이다.

#### SlugField

슬러그(slug)란 원래 언론사에서 제목을 쓸 때, 중요한 의미를 포함하는 단어만 사용해 제목을 작성하는 기법을 말한다. 예를 들면, 이 포스트의 마크다운 파일 이름은 '2018-6-22-model-field-types'인데 이것이 바로 슬러그이다. 이런 스타일의 슬러그는 워드프레스에서도 사용하며, URL에 이용함으로써 URL이 의미하는 바를 더 잘 표현할 수 있게 해준다.

SlugField는 이 슬러그를 위한 필드이며, 위와 같은 이유로 db_index=True 값을 기본으로 가지며 max_length의 기본값은 50이다. 숫자, 알파벳, '_', '-''만을 입력할 수 있다.

- **SlugField.allow_unicode** : True를 통해 유니코드 문자를 포함시킬 수 있다.

#### SmallIntegerField

PositiveSmallIntegerField와 같지만 범위가 -32768에서 32767이다.

#### TextField

매우 긴 문자열을 위한 필드이다. TextField에도 max_length를 지정할 수 있지만 데이터베이스까지 영향을 주지는 않는다. 그래야 할 경우에는 CharField를 사용해야 한다.

#### TimeField

파이썬의 datetime.time 인스턴스를 위한 필드이다. DateField와 같은 인자(auto_now, auto_now_add)들을 받을 수 있다.

#### URLField

URL을 위한 CharField이다. max_length의 기본값은 200이다.

#### UUIDField

[UUID]를 위한 필드이다. 이 필드는 파이썬의 UUID 클래스의 인스턴스를 값을 갖는다. UUID의 특성상 기본키로 사용하기에 아주 좋다.

[^1]: 이렇게 데이터베이스의 테이블-레코드 관계를 OOP의 클래스-인스턴스 관계로 바꾸어 관리하는 프로그래밍 기법(인터페이스)을 ORM(Object-Relational Mapping)이라고 부른다.

[^2]: IPv6 '::ffff:192.0.2.1'에서 IPv4 '192.0.2.1'을 언패킹할 수 있다.

[Model field reference 원문(Django Document)]: https://docs.djangoproject.com/en/2.0/ref/models/fields/
[UUID]: https://ko.wikipedia.org/wiki/%EB%B2%94%EC%9A%A9_%EA%B3%A0%EC%9C%A0_%EC%8B%9D%EB%B3%84%EC%9E%90
