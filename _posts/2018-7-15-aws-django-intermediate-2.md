---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 2. Django settings 설정"
date:   2018-7-15
categories: [aws]
---

<p class="intro"><span class="dropcap">A</span>WS 배포는 실제 사용환경으로 Django 프로젝트를 배포하는 것이기 때문에 몇몇 정보들을 은닉해야 한다. 특히 github 등의 오픈소스로 관리해야만 하는 경우에는 더욱 중요하다. 은닉을 위해 필요한 정보를 JSON으로 등록하고 settings 모듈의 파이썬 코드로 읽어들여야 한다.</p>

실제 사용환경으로 배포하기 위해서 점검해야할 사항을 정리해보면 아래와 같다.

1. DEBUG = False - 디버깅 정보를 오픈해버리면 소스를 그대로 보여주는 것과 다름이 없다. 실제 사용환경에서는 반드시 False로 설정해야 한다.
2. SECRET_KEY - 암호화 서명, 해싱 등에 사용하는 값이기 때문에 역시 노출하면 안된다.
3. ALLOWED_HOSTS - HTTP Host Header Attack 위협에 대응하기 위해서 ALLOWED_HOSTS 리스트 역시 비공개로 처리해야 한다.
4. RDS, S3 - AWS 계정 정보가 있기 때문에 비공개 처리한다.

이 중에서 4번의 RDS, S3 설정은 나중에 이어서 할 것이기 때문에 이는 제외하고 진행하려고 한다.

## JSON 활용하기

위의 4가지 정보를 저장하기 위해서 JSON을 활용하면 좋다. 먼저 프로젝트 루트 폴더에 ./.secrets 디렉토리를 만들고 secret.json 파일을 아래와 같이 만든다. 주의해야할 저은 "ALLOWED_HOSTS"의 값은 문자열이 아니라 리스트로 입력을 해야만 한다. (그렇지 않으면 Django에서 인식을 하지 못 한다.) 기본적으로 ".elasticbeanstalk.com"은 필요하다. EB에서 제공하는 URL로 바로 접속할 수 있기 때문이다.

```json
{
  "SECRET_KEY": "<Django secret key>",
  "ALLOWED_HOSTS": ["<Your list>"],
}
```

## .gitignore 추가

JSON으로 정보를 분리해봤자 github 등에 올라가버리면 도루묵이다. .gitignore에 ./.secrets 디렉토리를 추가하자. 매우 중요한 부분이니 꼭 확인해두자.

만약 실수로

## settings.py 파일 변경(DEBUG, ALLOWED_HOSTS)

파이썬에 JSON 파일을 읽어들이는 방법은 json 내장 라이브러리를 사용하면 된다. Django의 settings.py 파일에 아래와 같이 입력하면 된다.

```python
import json

ROOT_DIR = os.path.dirname(BASE_DIR)
SECRET_DIR = os.path.join(ROOT_DIR, '.secrets')
secrets = json.load(open(os.path.join(SECRET_DIR, 'secret.json'), 'rb'))

SECRET_KEY = secrets['SECRET_KEY']
DEBUG = False
ALLOWED_HOSTS = secrets['ALLOWED_HOSTS']
```

이제 최소한의 준비는 끝났다.

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
