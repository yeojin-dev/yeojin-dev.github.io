---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(기초편)"
date:   2018-7-3
categories: [aws]
---

<p class="intro"><span class="dropcap">A</span>WS(Amazon Web Service)는 미국 아마존사의 클라우드 서비스로 사실상 업계 표준이 된 상태이다. AWS에 가입하고 Django 기본 프로젝트를 AWS에 올려보았다.</p>

이 포스트에서는 단순히 AWS로 Django 프로젝트를 배포하는 것이 목표이다. Django 프로젝트 자체가 중요한 주제는 아니기 때문에 Django 기본 화면을 AWS를 통해 보여줄 것이다. 기초편이라 WSGI, Web Server 사용은 이 포스트에 나와있지 않다. 그것은 심화편 포스트를 통해서 정리하려고 한다.

## AWS 회원가입

AWS에 가입하는 것은 쉽다. AWS를 제대로 사용하려면 비용을 지불해야겠지만, 초보 수준에서는 1년간 무료로 이용할 수 있다. 아래 링크에서 AWS 회원가입을 하자. 해외 결제가 가능한 신용카드가 필수이다.

[AWS 회원가입](https://aws.amazon.com/free‎)


## EC2 설정

#### 1. 시작

가장 많이 이용하는 EC2(Elastic Compute Cloud)를 사용해보자. EC2는 AWS의 서버 컴퓨터를 일부분 임대해서 사용하는 것이라고 보면 된다. 먼저 아래 화면에서 EC2를 클릭하자.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-1.png' | prepend: site.baseurl }}" alt=""></div>

#### 2. 인스턴스 생성하기

클릭하면 나오는 화면에서 '인스턴스 시작' 버튼을 클릭하자. 여기서 인스턴스란 AWS의 서버 컴퓨터의 임대 단위를 말한다. 즉, 내가 사용하는 AWS의 컴퓨터 하나를 의미한다.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-2.png' | prepend: site.baseurl }}" alt=""></div>

#### 3. AMI 선택하기

'인스턴스 시작' 버튼을 클릭하면 AMI(Amazon Machine Image)를 선택해야 한다. 쉽게 말해서 어떤 운영체제 인스턴스를 사용할 것인지 선택하는 것인데 여기서는 가장 대중적인 우분투 리눅스를 사용해보자.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-3.png' | prepend: site.baseurl }}" alt=""></div>

#### 4. 인스턴스 유형 선택하기

우분투 리눅스를 선택했으면 인스턴스 유형을 선택해야 한다. 여기서 어느 규모로 AWS의 서버 컴퓨터를 임대할지 결정할 수 있는데 무료(프리 티어)는 고를 수 있는 것이 1개 뿐이다. 만약 비용을 지불할 수 있다면 더 좋은 환경에서 AWS를 이용할 수 있을 것이다.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-4.png' | prepend: site.baseurl }}" alt=""></div>

#### 5. 인스턴스 시작

프리 티어에서는 별도로 설정해야할 것이 없다. 아마 바로 단계 7로 넘어갈 겻이다. 바로 시작 버튼을 눌러서 인스턴스를 시작하자.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-5.png' | prepend: site.baseurl }}" alt=""></div>

#### 6. 인스턴스 키 페어 생성, 다운로드하기

인스턴스를 시작하기 전에 키 페어를 생성해야만 한다. 이는 보안을 위한 장치인데 이 키 페어를 통해야만 내 컴퓨터에서 원격으로 AWS EC2 인스턴스에 접속할 수 있다. 적당한 이름의 키 페어를 생성하고 다운로드하자. 여기서는 sample이라 이름지었다.

중요한 점은 키 페어는 1회 이상 다운로드가 불가능하다. 즉, 키 페어를 잃어버리면 다시는 인스턴스에 접속할 수 없다. 꼭 백업을 해두자.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-6.png' | prepend: site.baseurl }}" alt=""></div>

#### 7. 인스턴스 생성하기

이제 설정은 끝났고 인스턴스가 생성 중이다. 여기서 시간이 몇분 걸린다. 한편 sample.pem 파일을 다운로드했는지 확인하자. 방금 전에 얘기했듯이 백업을 해두어야만 한다.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-7.png' | prepend: site.baseurl }}" alt=""></div>

#### 8. 인스턴스 정보 확인하기

생성이 완료되면 대시보드가 나온다. (2번째 인스턴스는 필자가 이미 만들었던 인스턴스이니 무시해도 된다.) 먼저 1번째 인스턴스의 정보를 확인하자. 오른쪽 하단에 나온 퍼블릭 DNS(IPv4) 주소가 현재 인스턴스에 접속할 수 있는 주소이다. 만약 구입한 도메인이 있다면 연결할 수 있을 것이다.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-8.png' | prepend: site.baseurl }}" alt=""></div>

그리고 보안상의 이유로 인스턴스에 접속할 수 있는 포트에 대한 제한이 현재 설정되어있다. 기본값으로는 포트 22(ssh가 사용하는)로만 접속이 가능한데 임시로 Django 프로젝트에 접속가능하도록 설정해보자. 대시보드 좌측 하단의 보안 그룹을 클릭하자.

#### 9. 인스턴스 보안 그룹 설정하기

보안 그룹을 클릭하고 현재 인스턴스에 설정된 보안 그룹을 선택하자. (아마 되어 있을 것이다.) 그리고 하단의 인바운드 탭을 클릭하고 편집 버튼을 클릭한다.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-9.png' | prepend: site.baseurl }}" alt=""></div>

편집 버튼을 누르면 나오는 화면에서 아래와 같이 TCP 프로토콜의 8000 포트를 설정해주자.

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-10.png' | prepend: site.baseurl }}" alt=""></div>

그럼 이제 AWS에서의 설정은 끝났다! 이제 직접 AWS 서버에 들어갈 차례이다.

## EC2 인스턴스 ssh 접속

#### ssh로 원격 접속해보기

인스턴스에 원격 접속하기 위해서는 ssh를 사용해야 한다. 먼저 앞서 다운로드받은 pem 파일을 ~/.ssh 폴더에 옮겨놓자.

```shell
# mv ~/Download/sample.pem ~/.ssh
```

그럼 이제 인스턴스에 접속해보자. ssh 명령어를 사용하면 간단하게 들어갈 수 있다.

```shell
# ssh -i ~/.ssh/sample.pem ubuntu@ec2-13-125-229-43.ap-northeast-2.compute.amazonaws.com

Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-1060-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


# ubuntu@ip-172-31-22-185:~$
```

명령어에서 앞서 알게 된 퍼블릭 DNS(위 8번을 보자.) 주소를 붙여넣되 유저를 ubuntu로 설정해 접속했다는 것을 알 수 있다. 100% 같지는 않아도 대략 위와 같은 결과를 얻을 수 있을 것이다. 이렇게 하면 AWS 서버 컴퓨터의 어느 한 부분에 내가 원격으로 접속한 것이다!

#### Django를 위한 준비 (1) apt-get 업그레이드

현재 인스턴스에는 우분투 외에는 아무것도 설치되어 있지 않다. Django 프로젝트를 실행하기 위해서는 파이썬을 포함해 여러 패키지들을 설치해줘야 한다.[^1]

먼저 이 패키지를 관리해주는 리눅스 패키지 매니저 apt-get을 업그레이드 해주자.

```shell
# ubuntu@ip-172-31-22-185:~$ sudo apt-get update
# ubuntu@ip-172-31-22-185:~$ sudo apt-get dist-upgrade
```

#### Django를 위한 준비 (2) 인코딩 설정

UTF-8 인코딩 지원을 위해 locale 설정을 추가해야 한다.

```shell
# ubuntu@ip-172-31-22-185:~$ sudo vim /etc/default/locale
```

vim으로 locale 파일에 아래 3줄을 추가하자.

```shell
LC_CTYPE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"
```

여기까지 설정했으면 접속을 종료하고 재접속해야만 한다. ssh 접속을 종료할 때는 exit 명령어를 사용한다.

```shell
# ubuntu@ip-172-31-22-185:~$ exit
```


#### Django를 위한 준비 (3) pip, pipenv, pyenv 설치

다시 인스턴스에 접속했으면 pipenv 설치를 위해 pip를 설치하자.

```shell
# ubuntu@ip-172-31-22-185:~$ sudo apt-get install python-pip
# ubuntu@ip-172-31-22-185:~$ sudo python -m pip install -U pip
```

이어서 Django 프레임워크를 설치하기 위한 pipenv를 설치하자.

```shell
# ubuntu@ip-172-31-22-185:~$ sudo -H pip install -U pipenv
```

Django 프레임워크는 Python3 기반으로 동작하기 때문에 pyenv도 설치(requirements 포함)해야 한다.

```shell
# ubuntu@ip-172-31-22-185:~$ curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
# ubuntu@ip-172-31-22-185:~$ sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libffi-dev
```

설치를 순조롭게 마쳤으면 exit 후 다시 재접속하자.

#### Django를 위한 준비 (4) Python3 설치

pyenv를 설치했으니 Python3 설치는 매우 쉬울 것이다.[^2]

```shell
# ubuntu@ip-172-31-22-185:~$ pyenv install 3.6.5
```

이제 Django 프로젝트를 실행하기 위한 준비는 모두 마쳤다. 이제는 Django의 영역이다.

## Django 설정하기

이제 인스턴스를 나와서 내 컴퓨터로 돌아오자. 현재 상황에서 Django 프로젝트에 특별히 더 손댈 곳은 settings.py 하나뿐이다. 왜냐하면 Django 프로젝트가 서비스할 수 있는 호스트/도메인 설정을 해주어야 하기 때문이다. 현재 우리 EC2 인스턴스의 도메인은 'amazonaws.com'이기 때문에 settings.py의 ALLOWED_HOST 리스트를 아래와 같이 수정[^3]해준다.

```python
ALLOWED_HOSTS = [
    '.amazonaws.com',
]
```

## Django 프로젝트 인스턴스로 옮기기

이제 설정은 끝났다. 내가 만들어놓은 Django 프로젝트를 EC2 인스턴스로 옮기면 된다. 원격 파일 전송은 scp 명령어를 통해 가능하다.

```shell
# scp -i ~/.ssh/sample.pem -r ~/projects/django ubuntu@ec2-13-125-229-43.ap-northeast-2.compute.amazonaws.com:/home/ubuntu/project
```

scp 명령어의 내용을 간단하게 정리해보자.

- **-i** : 키 페어를 통해 접속
- **~/.ssh/sample.pem** : 해당 키
- **-r** : 재귀적으로 하위 디렉토리까지 모두 전송
- **~/projects/django** : 전송하고자 하는 디렉토리
- **ubuntu@ec2-13-125-229-43.ap-northeast-2.compute.amazonaws.com** : 퍼블릭 DNS(IPv4)
- **:/home/ubuntu/project** : EC2 인스턴스 내 디렉토리 주소

이렇게 하면 순조롭게 파일을 전송할 수 있다.

그럼 이제 EC2 인스턴스에서 Django 프로젝트를 실행시키면 된다.

## Django runserver in EC2 Instance

#### Django 프레임워크 설치

파일을 모두 옮겼으니 인스턴스에 원격 접속한다. 접속한 후에는 미리 설치해둔 pipenv를 통해 Django 프레임워크를 설치하자.

```shell
# ubuntu@ip-172-31-22-185:~/project$ pipenv install django
# ubuntu@ip-172-31-22-185:~/project$ pipenv shell
```

이제 직접 Django 프로젝트를 실행해보자. 보안 설정에서 포트 8000을 열었기 때문에 0:8000을 인자로 넣어주어야 한다.

```shell
(project-ipe16O64) ubuntu@ip-172-31-22-185:~/project$ python app/manage.py runserver 0:8000
```

그럼 끝!

<div style="text-align: center;"><img src="{{ '/assets/img/ec2-11.png' | prepend: site.baseurl }}" alt=""></div>

[^1]:필요에 따라 zsh, oh-my-zsh 등을 설치할 수도 있을 것이다. 하지만 필수적인 사항은 아니기 때문에 이 포스트에서는 제외했다. 자세한 내용은 [여기]에서 볼 수 있다.
[^2]:현재 가장 최신 버전인 3.7.0을 설치했다.
[^3]:이는 HTTP Host header attacks 위협에 대비하기 위함이다.

[여기]:https://subicura.com/2017/11/22/mac-os-development-environment-setup.html
