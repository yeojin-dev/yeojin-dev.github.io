---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 10. Django Log 확인 자동화하기"
date:   2018-7-26
categories: [aws]
---

<p class="intro"><span class="dropcap">E</span>B와 Docker를 이용해서 Django 애플리케이션을 배포하면 생기는 문제 중의 하나가 바로 로그 확인이다. 로그가 Docker 컨테이너 안에 존재하기 때문에 내 OS - EB - Docker 순으로 들어가야만 한다. 이를 편리하게 하기 위해서 스크립트를 작성해보자.</p>

먼저 Django 애플리케이션의 로그를 파일로 저장하는 방법은 [Django 로그 사용하기] 포스트를 확인하자.

## IDEA

포스트 내용대로라면 로그 파일은 `/var/log/django` 디렉토리에 저장될 것이다. 그런데 이 디렉토리는 EB 내에 있는 것이 아니라 EB가 실행하고 있는 Docker 컨테이너에 안에 저장되어 있다. 그렇기 때문에 우리는 다음의 3단계를 거쳐야만 한다.

1. EB에서 Docker 컨테이너 접속
2. Docker 컨테이너에서 로그 파일 EB로 복사해오기
3. EB에 있는 파일을 로컬 컴퓨터로 복사해오기
4. 복사해온 파일을 cat, tail 등의 명령어로 확인하기

여기서 가장 복잡한 것은 의외로 1번이다. 셸 스크립트를 통해서 구현하려면 awsebcli, docker 명령어를 잘 사용해야 한다. 먼저 프로필 이름으로 EB 환경 이름을 가져오고 환경에서 실행 중인 Docker 컨테이너의 ID를 가져온다. 그 후에는 쉽게 진행이 가능하다.

아래 셸 스크립트를 보자.

```shell
#!/usr/bin/env bash
EB_ENV=$(eb list --profile < profile name >)
EB_DOCKER_ID=$(eb ssh ${EB_ENV##* } --command "sudo docker ps -q")
EB_ERRORS_DIR=/var/log/django/
eb ssh ${EB_ENV##* } -c "sudo docker cp ${EB_DOCKER_ID:0:4}:/var/log/django/error.log ."  # Docker 컨테이너에서 EB로 파일 가져오기
mkdir ./.log
scp -i ~/.ssh/< pem file > -r ec2-user@< ec2 ip >:/home/ec2-user/error.log ./.log/error.log  # EB에서 로컬 컴퓨터로 파일 가져오기
eb ssh ${EB_ENV##* } -c "rm -rf error.log"  # EB에 있는 파일은 지우기
tail -10 ./.log/error.log
```

셸 스크립트에서 profile name, pem file, ec2 ip는 자신의 프로젝트에 맞게 설정해주어야 한다. 이 중에서 ec2 ip는 `eb ssh` 명령어를 사용해보면 쉽게 알아낼 수 있을 것이다.

## 마치며

간단하게 끝날 것 같았던 AWS 배포가 무려 10개의 포스트 작성이 필요한 큰 프로젝트가(?) 되었다. 이것만으로 끝이 아니겠지만 여기까지 완성하면 EB, S3, RDS까지 사용했으니 충분히 괜찮은 배포가 되었다고 생각한다. 더 고급 수준의 포스팅을 할 수 있기를 바라며... [지금까지 작업을 푸시한 깃허브 저장소]를 올려두었다. 포스팅하지는 않았지만 더 도움이 되는 코드도 있을 것이다. 아무쪼록 도움이 되었으면 좋겠다.

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

[지금까지 작업을 푸시한 깃허브 저장소]: https://github.com/FC-Oasis/baeminchan-django/commit/865f15b4a9a2ab2fd59a6baa5a9e94f704a8c7c5
