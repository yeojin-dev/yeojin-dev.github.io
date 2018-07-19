---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 6. S3 사용하기"
date:   2018-7-19
categories: [aws]
---

<p class="intro"><span class="dropcap">S</span>3 서비스는 AWS에서 제공하는 스토리지 서비스이다. 컴퓨팅 환경이 아니기 때문에 Django 프로젝트에서는 정적 파일을 저장하는 용도로 사용하기에 좋다.</p>

AWS S3를 생성하기 위해서는 boto3를 사용해서 S3 버킷을 생성해야 한다. 이 내용은 [이전 포스팅]에서 정리했으니 참고하자.

## djagno-storages, boto3 설치

Django는 settings.py의 DEFAULT_FILE_STORAGE, STATICFILES_STORAGE 값을 설정하지 않으면 Django 기본 스토리지 클래스인 django.core.files.storage.FileSystemStorage를 사용한다. S3를 이용하기 위해서는 이 두 값을 변경하기 위한 새로운 클래스가 필요하다.

클래스를 쉽게 만들어주도록 돕는 패키지가 'django-storages'이다. 그리고 이 패키지는 S3와의 통신을 위해 boto3 패키지를 필요로 한다. 그러므로 이 두 패키지를 설치하자.

```shell
$ pipenv install djang-storages boto3
```

설치를 마쳤으면 settings.py의 INSTLLED_APPS에 'storages'를 추가하자.

```python
INSTALLED_APPS += [
    'storages',
]
```

## S3BotoStorage 클래스 선언

이제 Django 프로젝트에서 S3를 사용하기 위한 클래스를 선언해야 한다. django-storages 패키지를 설치했으면 S3S3BotoStorage 클래스를 사용할 수 있으며 이 클래스를 상속받아서 원하는 Storage 클래스를 만들 수 있다.

이 클래스는 storages.py 파일에 선언했으며 정적 파일은 S3 버킷 내 "static"이라는 이름의 디렉토리에 누구나 접근 가능하고, 미디어 파일(사용자가 업로드한 파일 등)은 S3 버킷 내 "media"라는 이름의 디렉토리에 저장하되 인증받은 사용자만 접근할 수 있도록 설정했다.

```python
from storages.backends.s3boto3 import S3Boto3Storage

__all__ = (
    'S3StaticStorage',
    'S3DefaultStorage',
)


# 정적 파일용
class S3StaticStorage(S3Boto3Storage):
    default_acl = 'public-read'
    location = 'static'


# 미디어 파일용
class S3DefaultStorage(S3Boto3Storage):
    default_acl = 'private'
    location = 'media'
```

이제 이 클래스를 settings.py의 DEFAULT_FILE_STORAGE, STATICFILES_STORAGE 값으로 할당해주자. 클래스 경로를 문자열로 설정해주면 된다.

```python
DEFAULT_FILE_STORAGE = 'config.storages.S3DefaultStorage'
STATICFILES_STORAGE = 'config.storages.S3StaticStorage'
```

## AWS 관련 계정 설정하기

Django 스토리지 설정은 마쳤지만 AWS S3 접속 계정 관련 정보를 설정해주어야 한다. 이 정보는 중요한 정보이기 때문에 .secrets 디렉토리의 secret.json 파일에 저장하고 Django에서 읽어들여야 한다.

secret.json 파일에 아래 값들을 추가하자. 처음부터 여기까지 따라왔으면 SECRET_KEY, ALLOWED_HOSTS 값은 이미 있을 것이다.

```json
# .secrets/secret.json
"AWS_ACCESS_KEY_ID": "<AWS_ACCESS_KEY_ID>",
"AWS_SECRET_ACCESS_KEY": "<AWS_SECRET_ACCESS_KEY>",
"AWS_DEFAULT_ACL": "private",
"AWS_S3_REGION_NAME": "ap-northeast-2",
"AWS_S3_SIGNATURE_VERSION": "s3v4",
"AWS_STORAGE_BUCKET_NAME": "<AWS_STORAGE_BUCKET_NAME>"
```

secret.json의 정보를 settings.py에서 읽어오는 코드를 작성하자.

```python
# setting.py
AWS_ACCESS_KEY_ID = secrets['AWS_ACCESS_KEY_ID']
AWS_SECRET_ACCESS_KEY = secrets['AWS_SECRET_ACCESS_KEY']
AWS_DEFAULT_ACL = secrets['AWS_DEFAULT_ACL']
AWS_S3_REGION_NAME = secrets['AWS_S3_REGION_NAME']
AWS_S3_SIGNATURE_VERSION = secrets['AWS_S3_SIGNATURE_VERSION']
AWS_STORAGE_BUCKET_NAME = secrets['AWS_STORAGE_BUCKET_NAME']
```

## settings.py 설정

정적 파일, 미디어 파일이 Django 프로젝트 안에서 어디에 위치하고 있는지, 그리고 어떤 URL을 통해 접근해야 하는지 settings.py에 설정을 해야한다. 관련 변수 이름은 아래와 같다.

1. STATIC_URL : 정적 파일에 접근할 URL 접두어
2. MEDUA_URL : 미디어 파일에 접근할 URL 접두어
3. STATICFILES_DIRS : Django 프로젝트 내에 정적 파일들이 저장된 경로들(리스트). manage.py의 collectstatic 명령어로 정적 파일을 수집할 때 사용하는 경로.

settings.py 파일에 아래와 같이 코드를 써주자.

```python
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
```

위와 같이 설정하면 'Root URL(로컬 호스트라면 localhost:8000/) + STATIC_URL'의 경로로 정적 파일에 접근할 수 있게 된다. 이 경로는 S3 버킷과 연결이 될 것이며, S3 버킷에는 collectstatic 명령어에 의해 STATICFILES_DIRS에 설정된 경로의 파일들이 모여있을 것이다.

그렇다면 어떻게 STATIC_URL의 주소와 S3 버킷이 연결될까? 이를 위해서는 nginx 설정을 바꿔야 한다.

## nginx_app.py 설정

[AWS로 Django 프로젝트 배포하기(중급) 3. nginx, uwsgi, supervisor 설정] 포스팅에서 nginx_app.conf 파일을 만들었다. 다시 그 파일을 들여다보자.

```nginx
server {
    listen 7000;
    charset UTF-8;
    server_name *.elasticbeanstalk.com;
    client_max_body_size 128M;

    location / {
        uwsgi_pass  unix:///tmp/app.sock;
        include     uwsgi_params;
    }
}
```

여기서 location을 보면 모든 URL 요청에 대해서 uwsgi와 소켓 연결을 하도록 되어 있다. 즉 Django 애플리케이션으로 연결되고 있다. 그런데 정적 파일, 미디어 파일은 웹 애플리케이션에서 별도의 처리를 할 것이 없다. 있는 파일 그대로 전송해주면 되기 때문이다. 그러므로 정적 파일, 미디어 파일은 Django 애플리케이션이 아닌 S3 버킷과 직접 연결되어야 한다.

여기서 이미 위의 settings.py 파일에서 정적 파일은 '/static/', 미디어 파일은 '/media/' URL을 통해 연결하도록 설정해두었다. 그러므로 이 URL에 대하여 요청이 오면 nginx가 S3 버킷으로 요청을 보내도록 설정을 하면 된다. S3StaticStorage, S3DefaultStorage 클래스를 만들 때 location을 설정하였기 때문에 이를 반영해서 nginx_app.conf 파일을 설정한다.

```nginx
server {
    listen 7000;
    charset UTF-8;
    server_name server.yeojin.me;
    client_max_body_size 128M;

    location / {
        uwsgi_pass  unix:///tmp/app.sock;
        include     uwsgi_params;
    }

    location /static/ {
        alias           https://<S3 버킷 URL>/static/;
    }

    location /media/ {
        alias           https://<S3 버킷 URL>/media/;
    }
}
```

\<S3 버킷 URL\>은 각자 만든 S3 버킷의 URL 주소로, AWS 콘솔에서 확인할 수 있다. 그리고 위 파일에서 `location /static/`, `location /media/` 부분은 settings.py 파일의 STATIC_URL, MEDTA_URL에 의한 것이며, alias 부분은 S3DefaultStorage.location, S3StaticStorage.location 설정과 관련이 있음을 알 수 있다.

## collectstatic

이제 collectstatic 명령어를 실행하면 Djagno 애플리케이션은 프로젝트 내 정적 파일들(STATICFILES_DIRS로 설정한 경로 내 파일들)을 수집해 S3에 저장할 것이다.

```shell
$ python manage.py collectstatic
```

이제 Django Admin 페이지를 들어가면 CSS가 반영된 화면이 나올 것이다. 하지만 아직 모든 문제가 해결되지 않았다. CSS 내 폰트 사용 등에서 CORS 문제가 발생할 것이기 때문이다. 자세한 것은 브라우저의 개발자 모드에서 콘솔 부분을 보면 알 수 있다. 이것은 Django Admin 페이지의 자바스크립트에서 폰트 파일을 불러오기 때문이다.

## CORS 문제 해결

CORS 문제가 무엇인지는 [이 포스팅]을 참고하자. 현재 우리 상황으로 설명하자면 EB의 도메인과 S3 버킷이 서로 다르기 때문에 CORS 문제가 발생한다. HTML 문제에서 직접적으로 크로스 도메인 요청을 하는 것은 문제가 없지만, HTML 내부의 자바스크립트에서 요청한다면 문제가 발생한다. 이를 해결하기 위해서는 S3 버킷의 CORS 정책을 수정해주어야만 한다.

![S3 CORS](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/images/s3-cors-config.png)

여기서 아래와 같이 추가해주자.

```xml
<CORSRule>
  <AllowedOrigin><내 도메인></AllowedOrigin>
  <AllowedMethod>GET</AllowedMethod>
  <MaxAgeSeconds>3000</MaxAgeSeconds>
</CORSRule>
```

\<내 도메인\> 부분은 내 EB 도메인을 입력해주면 된다. 위와 같이 설정하면 내 도메인에서의 크로스 사이트 요청은 문제를 일으키지 않을 것이다.

#### AWS로 Django 프로젝트 배포하기(중급) 포스트 링크

1. [EB 인스턴스 만들기](https://yeojin-dev.github.io/blog/aws-django-intermediate-1/)
2. [Django settings 설정](https://yeojin-dev.github.io/blog/aws-django-intermediate-2/)
3. [nginx, uwsgi, supervisor 설정](https://yeojin-dev.github.io/blog/aws-django-intermediate-3/)
4. [Dockerfile 설정](https://yeojin-dev.github.io/blog/aws-django-intermediate-4/)
5. [Docker 컨테이너 EB 인스턴스 배포](https://yeojin-dev.github.io/blog/aws-django-intermediate-5/)
6. [S3 사용하기](https://yeojin-dev.github.io/blog/aws-django-intermediate-6/)
7. [RDS 사용하기](https://yeojin-dev.github.io/blog/aws-django-intermediate-7/)
8. [Django Log 사용하기](https://yeojin-dev.github.io/blog/aws-django-intermediate-8/)
9. [ELB Health Check 통과하기](https://yeojin-dev.github.io/blog/aws-django-intermediate-9/)
10. [Django Log 확인 자동화하기](https://yeojin-dev.github.io/blog/aws-django-intermediate-10/)

[이전 포스팅]:https://yeojin-dev.github.io/blog/s3-bucket-by-python/
[AWS로 Django 프로젝트 배포하기(중급) 3. nginx, uwsgi, supervisor 설정]:https://yeojin-dev.github.io/blog/aws-django-intermediate-3/
[이 포스팅]:https://homoefficio.github.io/2015/07/21/Cross-Origin-Resource-Sharing/
