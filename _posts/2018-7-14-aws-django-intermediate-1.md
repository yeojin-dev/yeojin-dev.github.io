---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 1. EB 인스턴스 만들기"
date:   2018-7-14
categories: [aws]
---

<p class="intro"><span class="dropcap">E</span>lastic Beanstalk는 AWS의 PaaS이다. EC2에 비해서 로드 밸런싱 등 편리하게 배포할 수 있다는 점에서 유용하다. 특히 Docker를 지원하기 때문에 더더욱 그렇다.</p>

이번 AWS 배포 포스팅은 내용이 꽤 길 것으로 예상된다. 이전에 블로그에 올렸던 [EC2로 배포하기] 포스팅보다 더 복잡한 구조로 배포할 것이기 때문이다. EC2만으로 배포를 한다면 DB 관리나 스토리지 관리 등에서 단점이 큰 반면, EB, RDS, S3로 연계해서 배포하는 것은 서버, DB, 스토리지를 분리하기 때문에 안정성이 높다. 아마 이렇게 배포하는 것이 익숙해진다면 이전의 EC2만으로 배포하는 방법은 사용하지 않을 것이다.

## IAM 사용자 계정 만들기

EB 인스턴스를 만들기 위해서 IAM 사용자 계정을 만들 것이다. IAM 사용자 계정을 만드는 것은 간단하다. [이곳]의 포스팅을 참고하되, '기존 정책 직접 연결' 부분에서 EB 관리 권한이 있는 AWSElasticBeanstalkFullAccess 권한을 선택하자. 참고로 이 권한은 매우 강력한 권한이기 때문에 관련 값이 깃허브 등에 올라가지 않도록 유의해야 한다. [이런 일]을 당할 수 있다...

계정을 만들고 ~/.AWS/credentials 파일에 계정 정보를 저장하는 것까지 완료하자.

## Elastic Beanstalk 생성하기

#### awsebcli 설치

IAM 계정까지 만들었으면 이제 콘솔에서 파이썬을 이용해 EB 인스턴스를 만들어보자. 이를 위해서 awsebcli 패키지 설치가 필요하다. awsebcli 환경은 배포 환경과는 연관이 없기 때문에 --dev를 설정한다.

```shell
pipenv install awsebcli --dev
```

#### EB 애플리케이션 생성

awsebcli를 설치했으면 인스턴스를 생성해보자. Django 프로젝트 디렉토리에서 awsebcli 초기화를 하자. ~/.AWS/credentials 디렉토리에 저장된 IAM 계정 이름이 필요하다.

```shell
eb init --profie <profile name>
```

입력을 하면 리전, 사용할 애플리케이션(기존 앱 선택 또는 새로 생성), 플랫폼(Docker 선택) 등의 질문을 하는데 대답을 입력하면 EB 애플리케이션 생성이 완료된다.

#### EB 환경 생성

이제 애플리케이션 환경을 설정한다.

```shell
eb create
```

위 명령어를 입력하고 이름 외에 로드 밸런서 타입을 물어보는데 애플리케이션을 선택하면 된다.

#### 생성 확인

생성이 완료되면 프로젝트 디렉토리에 ./.awselasticbeanstalk 디렉토리가 생기고 config.yml 파일이 생성된다. 이 파일에서 아래와 같이 생성 정보를 확인할 수 있다.

```yml
branch-defaults:
  master:
    environment: <EB 환경 이름>
    group_suffix: null
global:
  application_name: <EB 애플리케이션 이름>
  branch: null
  default_ec2_keyname: <pem 파일 이름>
  default_platform: Docker 18.03.1-ce
  default_region: ap-northeast-2
  include_git_submodules: true
  instance_profile: null
  platform_name: null
  platform_version: null
  profile: <IAM 계정 이름>
  repository: null
  sc: git
  workspace_type: Application
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

[EC2로 배포하기]:https://yeojin-dev.github.io/blog/aws-django-basic/
[이곳]:https://yeojin-dev.github.io/blog/s3-bucket-by-python/
[이런 일]:http://sanghaklee.tistory.com/32
