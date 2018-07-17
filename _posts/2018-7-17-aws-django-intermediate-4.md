---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 4. Dockerfile 설정"
date:   2018-7-17
categories: [aws]
---

<p class="intro"><span class="dropcap">E</span>B 플랫폼 중 하나인 Docker는 가상화 플랫폼이다. 가상화를 했기 때문에 로컬에서 Docker 컨테이너가 문제없이 작동한다면 EB에서도 문제없이 작동함을 기대할 수 있다.</p>

이전 포스팅까지 기본적인 설정들을 마쳤다. 아직 DB도 SQLite이고 정적 파일들을 위한 스토리지도 없지만 EB에서 작동하긴 할 것이다. 이제 지금까지 준비한 것들이 Docker 플랫폼에서 작동하도록 Docker 컨테이너를 만들어야 한다. 컨테이너는 Docker 이미지를 통해서 만들어야 하는데 이 이미지를 생성하는 파일이 Dockerfile이다. (이 파일은 확장자가 없음에 주의하자.)

만약 Docker에 대해 궁금하다면 [subicura님의 글 "초보를 위한 도커 안내서 - 도커란 무엇인가?"]를 추천한다. 충분히 일독할 만한 가치가 있는 글이다.

## Dockerfile (1)

먼저 Docker 컨테이너는 리눅스 환경임을 알아두자. Docker 컨테이너가 먼저 가지고 있어야 하는 환경을 생각해보자.

1. 리눅스와 apt 사용 환경
2. 리눅스 설치 프로그램 : nginx, supervisor
3. 파이썬 : 여기서는 3.6.5 버전을 사용한다. Django 개발환경과 같은 버전이면 된다.
4. 파이썬 패키지 : django, uwsgi

여기서 4번 항목의 패키지 리스트는 pipevn, pyenv를 사용한다면 requirements.txt 파일을 통해서 설치할 수 있을 것이다. 이 포스팅에서는 pipenv를 사용했기 때문에 `pipenv lock --requirements > requirements.txt` 명령어로 생성할 수 있다.

그럼 위 내용들을 포함하는 Dockerfile을 만들어보자. Dockerfile은 Dockerfile 생성 양식에 따라 만들면 된다.

```dockerfile
FROM        python:3.6.5-slim
MAINTAINER  yeojin.dev@gmail.com

RUN         apt -y update && apt -y dist-upgrade
RUN         apt -y install build-essential
RUN         apt -y install nginx supervisor

COPY        ./requirements.txt /srv
RUN         pip install -r /srv/requirements.txt
```

위 파일을 보면 Docker에서 제공하는 기본 이미지인 python:3.6.5-slim을 이용한다. 이 이미지는 데비안 계열 리눅스에 파이썬이 설치된 이미지이다. 데비안이므로 패키지 관리자는 apt를 사용한다. 그래서 apt를 최신 버전으로 업데이트한 후 nginx, supevisor를 설치한다.

그리고 내 컴퓨터의 루트 폴더에 존재하는 requirements.txt 파일을 Docker 이미지의 /srv 디렉토리에 복사한 후 requirements.txt 파일에 기록된 파이썬 패키지들을 설치하고 있다.

## Dockerfile (2)

이제 해야할 일은 Django 프로젝트 및 기타 설정 파일들을 옮겨야 하는 것이다. 위 Dockerfile에 이어서 아래 내용을 입력하자.

```dockerfile
ENV         PROJECT_DIR              /srv/project

COPY        .                       ${PROJECT_DIR}
WORKDIR     ${PROJECT_DIR}

RUN         cp -f ${PROJECT_DIR}/.config/nginx.conf           /etc/nginx
RUN         cp -f ${PROJECT_DIR}/.config/nginx_app.conf       /etc/nginx/sites-available

RUN         rm -f /etc/nginx/sites-enabled/*
RUN         ln -fs /etc/nginx/sites-available/nginx_app.conf                    /etc/nginx/sites-enabled

RUN         cp -f ${PROJECT_DIR}/.config/supervisor_app.conf  /etc/supervisor/conf.d

EXPOSE      7000

CMD         supervisord -n
```

위 내용을 정리해보자.

1. 내 컴퓨터의 루트 폴더에 존재하는 모든 파일(Django 프로젝트, .secret, .config 디렉토리)을 컨테이너의 /srv/project 디렉토리에 복사
2. 이 중 nginx.conf 파일은 /etc/nginx 디렉토리로 복사
3. 마찬가지로 nginx_app.conf 파일은 /etc/nginx/sites-available 디렉토리로 복사
4. 컨테이너의 /etc/nginx/sites-available/nginx_app.conf 파일에 대한 /etc/nginx/sites-enabled 디렉토리로 링크 연결
5. supervisor_app.conf 파일을 /etc/supervisor/conf.d 디렉토리로 복사
6. 7000번 포트 개방
7. supervisor 실행 - supervisord -n 명령어로 데몬으로 실행

여기서 2~4번 내용은 nginx의 설정과 관련이 있다.

마지막 supervisord -n 명령어를 통해 nginx와 uwsgi가 연결되고 uwsgi는 nginx와 Django 애플리케이션을 연결해준다.

그리고 이 모든 것들이 Docker 컨테이너 안에서 이루어진다.

그럼 이제 이 컨테이너를 EB 인스턴스를 통해 실행시키면 된다. 이 배포 과정은 다음 포스팅에서 이어서 하겠다.

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

[subicura님의 글 "초보를 위한 도커 안내서 - 도커란 무엇인가?"]:https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html
