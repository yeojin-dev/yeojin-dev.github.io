---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 9. ELB Health Check 통과하기"
date:   2018-7-25
categories: [aws]
---

<p class="intro"><span class="dropcap">E</span>LB(Elastic Load Balancer)는 AWS에서 제공하는 로드 밸런싱 서비스이다. 로드 밸런싱이란 서버 시스템에 가해지는 부하를 여러 대의 시스템으로 분산해서 규모있는 시스템을 만들어주는 기술이다. ELB Health Check는 ELB가 각 EC2의 상태를 확인하는 작업이다. Health Check를 통해서 로드 밸런서는 EC2가 부하를 분배할 수 있는 상태인지 아닌지 알 수 있다.</p>

실제 서버 배포에서는 보안상 ALLOWED_HOSTS를 통해 지정한 호스트만 서버에 접근할 수 있도록 해야 한다. 그렇다면 ELB Health Check를 위해서 해당하는 IP를 ALLOWED_HOSTS에 추가해주면 쉽게 끝나는 일인데 이것이 간단하지가 않다. 아래 도표를 보자.

![ELB](https://i2.wp.com/jayendrapatil.com/wp-content/uploads/2016/04/screen-shot-2016-04-05-at-7-54-34-am.png?resize=768%2C590)

ELB의 대략적인 역할을 알 수 있는 도표이다. ELB는 각 EC2에 Health Check를 보내는데 ELB와 EC2는 AWS의 로컬 네트워크 안에 존재한다. 즉, ELB와 EC2 사이에사는 로컬 IP를 가지고 통신을 하기 때문에 nginx 설정 등으로 ELB Health Check를 관리하는 것은 어렵다. Django 애플리케이션 내부에서 EC2의 로컬 IP를(이 IP가 HTTP 헤더의 호스트 이름으로 들어감) 알아낼 수 있어야 한다.

이를 위해서 [AWS 문서]를 보자. 문서에 따르면 `http://169.254.169.254/latest/meta-data/local-ipv4`에서 DNS 호스트 IP를 얻을 수 있다. 이 IP가 ELB의 IP이기 때문에 Django 애플리케이션 내부에서 이 URL에 접근해서 IP 주소를 동적으로 얻어내고 그 값을 ALLOWED_HOSTS에 추가해주어야 한다.

## Implementation

```python
response = urlopen('http://169.254.169.254/latest/meta-data/local-ipv4')
ec2_ip = response.read().decode('utf-8')
if response:
    response.close()
ALLOWED_HOSTS.append(ec2_ip)
```

코드는 간단하지만 실제로는 이렇게 간단하지가 않다. 위 코드는 EC2 시스템에서 실행하지 않는 경우라도 요청을 보내기 때문이다. 그러므로 Django 애플리케이션이 EC2 시스템에서 동작하는지 알기 위해서 [UUID]를 확인하는 코드를 함께 작성해야 한다. UUID는 `/sys/hypervisor/uuid` 파일에서 확인할 수 있다.

```python
def is_ec2_linux():
    if os.path.isfile('/sys/hypervisor/uuid'):
        with oepn('/sys/hypervisor/uuid') as f:
          uuid = f.read()
          return uuid.startswith('ec2')
    return False
```

위와 같은 코드와 함께 사용하면 ELB Health Check를 통과할 수 있을 것이다.

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

[AWS 문서]: https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-instance-metadata.html
[UUID]: https://ko.wikipedia.org/wiki/%EB%B2%94%EC%9A%A9_%EA%B3%A0%EC%9C%A0_%EC%8B%9D%EB%B3%84%EC%9E%90
