---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 7. RDS 사용하기"
date:   2018-7-23
categories: [aws]
---

<p class="intro"><span class="dropcap">R</span>DS는 AWS에서 제공하는 클라우드 데이터베이스 서비스이다. AWS EB와 연계해서 사용하면 DB 관리와 웹 앱 관리를 나누어서 처리할 수 있다.</p>

## RDS 생성

AWS에서 제공하는 문서 [1단계: RDS DB 인스턴스 만들기]에서 DB 인스턴스를 만들자. 다만 Django와 가장 연계가 좋은(우선적으로 지원하는) PostgreSQL 인스턴스를 생성하면 된다. 생성하면서 아래 정보를 기억해두자.

1. HOST : ".ap-northeast-2.rds.amazonaws.com" 형식의 도메인
2. Master Username : RDS 내 DB의 관리자 이름
3. Master Password : 관리자 비밀번호
4. Database Name : RDS 내에 생성된 DB 이름

만약 RDS를 생성했을 때 DB를 만들지 않았다면 아래에 나와 있는 'RDS 직접 접속하기' 방법으로 RDS에 들어가서 생성하면 된다.

## 패키지 설치

RDS에 접속해서 PostgreSQL CLI를 처리할 수 있는 패키지가 필요하다.

```shell
$ brew install postgresql
$ pipenv install psycopg2-binary
```

## sercret.json, settings.py 설정

RDS와 관련된 AWS 정보를 Django settings.py 파일에 입력해야하기 때문에 .secret 디렉토리의 secret.json 파일에 아래와 같이 JSON 정보를 입력하자. [Django settings 설정](https://yeojin-dev.github.io/blog/aws-django-intermediate-2/) 포스팅을 참고하자.

```json
"DATABASES": {
  "default": {
    "ENGINE": "django.db.backends.postgresql",
    "HOST": "<DB host>",
    "PORT": "5432",
    "USER": "<DB user>",
    "PASSWORD": "<DB pw>",
    "NAME": "<DB name>"
  }
```

HOST, USER, PASSWORD, NAME에 위에서 설정한 정보를 입력해주면 된다.

## RDS 접속해보기

만약 \<DB name\>을 모른다면 어떻게 해야 할까?
RDS에 접속하는 방법은 간단하다.

```shell
$ psql --host=<DB host> --user=<DB user> --port=5432 postgres
```

명령어에 호스트와 관리자 이름을 입력하면 된다. postgres는 PostgreSQL의 기본 관리자 롤(Role)을 의미한다. 접속후 `\l` 명령어로 데이터베이스 목록을 확인할 수 있으며 데이터베이스가 없다면 `CREATE DATABASE` 명령어를 이용해서 데이터베이스를 생성하면 된다.

만약 RDS에 접속이 잘 되었고 데이터베이스 이름도 확인했다면 마이그레이션을 진행한다. 만약 마이그레이션이 문제 없이 진행되었다면 RDS를 EB에 연결하는 것 역시 완료된 것이다.

```shell
$ ./manage.py migrate
```

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
