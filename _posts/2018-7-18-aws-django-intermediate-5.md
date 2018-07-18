---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 5. Docker 컨테이너 EB 인스턴스 배포"
date:   2018-7-18
categories: [aws]
---

<p class="intro"><span class="dropcap">D</span>ocker 컨테이너를 통해 nginx, uwsgi, supervisor가 동작하는 가상환경을 만들었다. 이제 이 컨테이너를 EB 인스턴스에 배포해서 실제로 접속할 수 있도록 해보자.</p>

## DEBUG, ALLOWED_HOSTS 설정

EB 인스턴스에 배포하기 전에 DEBUG, ALLOWED_HOSTS 설정을 변경해야 한다. 배포한다는 것은 실제 인터넷 환경에 노출된다는 것을 의미하기 때문에 온갖 해킹 시도에 노출된다는 것을 의미한다. 그렇기 때문에 최소한의 설정을 하겠다.

DEBUG = True 상태에서는 HTTP 요청의 Host Header에 상관없이 요청을 처리한다. 이 경우 HTTP Host Header Attack에 노출될 수 있기 때문에 위험하다. 이를 방지하기 위해서 DEBUG = False 설정이 필요하고 이 설정에 따라 ALLOWED_HOSTS 리스트를 settings.py에 입력해주어야만 한다. 일종의 호스트 화이트 리스트라고 보면 된다. ALLOWED_HOSTS는 이미 [Django settings 설정](https://yeojin-dev.github.io/blog/aws-django-intermediate-2/)에서 설정했으니 생략하고 DEBUG만 값을 바꾸자.

```python
DEBUG = False
```

위 코드를 Django 프로젝트의 settings.py 파일에 입력하자. 이미 `DEBUG = True` 코드가 있을 것이니 그 부분을 바꿔주면 된다.

## eb deploy

awsebcli 패키지에서 지원하는 명령어가 eb deploy 명령어이다. 이 명령어는 Docker 플랫폼에서 현재 디렉토리의 Dockerfile을 찾아 Docker 컨테이너를 만들고 이 컨테이너를 설정한 EB 애플리케이션의 환경에 배포한다.

기억해야만 하는 점은 awsebcli는 현재 디렉토리의 저장된 내용을 기준으로 하는 것이 아니라 가장 최근 커밋을 기준으로 한다는 점이다. 즉 eb depoly를 실행하기 이전에 커밋을 끝까지 해두는 것이 중요하다.

그런데 여기서 중요한 점이 있는데 그것은 바로 .secrets 디렉토리이다. 우리는 [Django settings 설정](https://yeojin-dev.github.io/blog/aws-django-intermediate-2/)에서 .secrets 디렉토리를 .gitignore에 저장했다. 그렇기 때문에 .secrets는 커밋에 반영이 안되고 awsebcli에도 반영이 되지 않는다. 그래서 Dockerfile로 컨테이너를 만들 때 컨테이너로 복사가 되지 않는다. 그런데 .secrets에는 Django의 SECRET_KEY가 있기 때문에 이 디렉토리가 없으면 Django 애플리케이션을 실행할 수 없다.

어떻게 해야만 할까?

이럴 때를 위해서 eb deploy에서 활용하는 옵션이 --staged 옵션이다. 이 옵션은 가장 최근 커밋과 더불어 staged 상태의 파일까지 포함해 배포를 한다. 즉 .secrets 디렉토리의 내용을 커밋하지 않고 배포하는데 활용할 수 있다.

이상의 내용을 정리해서 최종적으로 우리가 사용할 명령어는 아래와 같다. requirements.txt 파일이 루트 폴더에 존재하는지 한 번 더 확인해보자.

```shell
$ git add -A
$ git add -f .secrets/
$ eb deploy --profile <your profile> --staged
$ git reset HEAD .secrets/
```

가장 마지막의 git reset을 확인하자. .secrets 디렉토리는 절대 커밋이 되어서는 안된다. 배포 이후에는 다시 제외시켜야만 할 것이다.

## 배포 이후의 문제들

우여곡절 끝에 배포가 완료되었지만 이것은 정말로 최소한의 배포이다. 로컬 환경과 프로덕션 환경 분리조차 되어있지 않다. 이 포스팅에서는 일단 EB에 올라가는 것만을 목목표로 했기 때문이다. 어쨌든 현재 상황에서 아래와 같은 문제들이 남아있다.

1. Django 프로젝트에 아무 것도 없기 때문에 직접 들어가면 "Not Found"가 나올 것이고
2. "/admin" URL로 접근하면 로그인 화면이 나오겠지만 CSS가 반영되어 있지 않다.
3. 지금 드러나 있지는 않지만 DB 역시 SQLite이기 때문에 성능이 좋지 않다.
4. DEBUG = False 이기 때문에 에러가 났을 때 로그를 확인할 수 없다.
5. ELB Health Check를 통과하지 못 하고 있다.

이 정도 문제를 생각해볼 수 있는데 여기서 1번은 Django 프로젝트의 문제이다. (하지만 5번 문제와 연관이 있다.) 적절한 url 설정과 view 함수를 만들어주면 자연스럽게 해결될 것이다. 그럼 2번부터 하나식 차근차근 해결해보자.

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
