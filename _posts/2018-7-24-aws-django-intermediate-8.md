---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 8. Django Log 사용하기"
date:   2018-7-24
categories: [aws]
---

<p class="intro"><span class="dropcap">D</span>EBUG = False 상태에서는 자세한 로그를 볼 수가 없다. 배포 시에는 DEBUG=False 설정이 필수적인데 서버 에러에 따른 로그를 확인하기 위해서는 별도의 파이썬 코드가 필요하다.</p>

이하 코드들은 모두 Django 프로젝트 내 settings.py에 저장해야 한다.

## LOG 디렉토리 설정

```python
LOG_DIR = '/var/log/django'
if not os.path.exists(LOG_DIR):
    LOG_DIR = os.path.join(ROOT_DIR, '.log')
    os.makedirs(LOG_DIR, exist_ok=True)

subprocess.call(['chmod', '755', LOG_DIR])
```

지금 AWS EB의 플랫폼은 Docker이기 때문에 Docker 리눅스 플랫폼 내 로그 저장 디렉토리인 `/var/log` 디렉토리에 로그를 저장하기 위한 디렉토리를 만들었다. 만약 폴더가 없다면 Django 디렉토리 내부에 '/.log' 디렉토리를 생성했다. 그리고 실행 권한을 주기 위해서 subprocess 내장 라이브러리를 사용했다.

## LOGGING

[Django Logging] 문서를 보면 로그를 위해서 Loggers, Handlers, Filters, Formatters를 딕셔너리 형태로 설정해주어야 한다.

아래 코드를 보자.

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse',
        },
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'formatters': {
        'django.server': {
            'format': '[%(asctime)s] %(message)s',
        }
    },
    'handlers': {
        'console': {
            'level': 'INFO',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
        },
        'file_error': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'ERROR',
            'formatter': 'django.server',
            'backupCount': 10,
            'filename': os.path.join(LOG_DIR, 'error.log'),
            'maxBytes': 10485760,
        }
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file_error'],
            'level': 'INFO',
            'propagate': True,
        },
    }
}
```

코드를 보면 서버 에러 발생시 `[%(asctime)s] %(message)s`에 맞게 로그를 생성한다. 콘솔에는 INFO 수준의 정보까지 보여주고 ERROR 수준의 로그가 나오면 파일로 기록한다. 로그 파일의 이름은 `error.log`이며 파일 크기가 10485760 바이트 이상이 되면 새로운 로그 파일을 생성해 기록한다.

## sentry

[sentry] 패키지를 활용하면 로그를 이메일로 받아보는 것도 가능하다. 이는 추후에 포스팅을 할 예정이다.

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

[1단계: RDS DB 인스턴스 만들기]: https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/CHAP_Tutorials.WebServerDB.CreateDBInstance.html
[Django Logging]: https://docs.djangoproject.com/en/2.0/topics/logging/#topic-logging-parts-loggers
[sentry]: https://pypi.org/project/sentry/
