---
layout: post
title:  "파이썬으로 AWS S3 버킷 만들기"
date:   2018-7-10
categories: [aws]
---

<p class="intro"><span class="dropcap">S</span>3는 AWS에서 제공하는 클라우드 스토리지 서비스이다. S3 서비스를 이용하기 위해서는 AWS 홈페이지의 콘솔에서 직접 버킷을 생성할 수 있지만 파이썬으로도 가능하다.</p>

파이썬으로 AWS에 접속해 버킷을 생성하기 위해서는 boto3이라는 파이썬 패키지를 설치하여야 한다. 자세한 내용은 [boto3 Document]를 참고하면 된다.

## IAM 사용자 계정 만들기

당연한 말이겠지만 AWS에 접속하기 위해서는 별도의 계정을 만들어야 한다. AWS에서는 IAM 서비스를 이용해 사용자 계정을 만들면 된다. IAM 메인 웹페이지에 접속해서 계정을 만들어보자. 적당한 계정 이름을 만들고 프로그래밍 방식 액세스를 체크하자.

<div style="text-align: center;"><img src="{{ '/assets/img/s3-1.png' | prepend: site.baseurl }}" alt=""></div>

그 다음에 나오는 암호 설정 부분은 '기존 정책 직접 연결' 부분을 클릭해 AmazonS3FullAccess 정책을 사용하자. 우리는 S3 버킷을 사용할 것이기 때문이다. 모든 권한을 주는 것보다 적절히 권한을 제한하는 것이 좋다.

<div style="text-align: center;"><img src="{{ '/assets/img/s3-2.png' | prepend: site.baseurl }}" alt=""></div>

검토 부분은 바로 넘어간다.

<div style="text-align: center;"><img src="{{ '/assets/img/s3-3.png' | prepend: site.baseurl }}" alt=""></div>

그 다음으로 액세스 키 ID와 비밀 액세스 키를 알려주는데 이는 잘 관리해야만 한다. 이 키가 외부에 노출될 경우 심각한 피해를 입을 수 있다. 이 두 값은 나중에 저장해야하니 적당한 곳에 복사해두자.

<div style="text-align: center;"><img src="{{ '/assets/img/s3-4.png' | prepend: site.baseurl }}" alt=""></div>

## config, credentials 작성

파이썬으로 접속하기 위해서 config, credentials 파일의 작성이 필요하다. 이 두 파일인 확장자가 없으며 ~/.aws 디렉토리에 위치한다. VIM을 이용해 아래와 같이 파일을 만들자.

1. ~/.aws/config

```
[default]
region=ap-northeast-2
output=json
```

ap-northeast-2는 AWS 서울 리전을, json은 생성 결과를 json 형식으로 받는다는 것을 의미한다.

2. ~/.aws/credentials

```
[sample-profile-name]
aws_access_key_id=AAAAAAAAAAAAA
aws_secret_access_key=BBBBBBBBBBBBBBBBBBBBBBB
```

aws_access_key_id, aws_secret_access_key는 IAM에서 사용자 계정을 만들 때 복사해둔 액세스 키 ID, 비밀 액세스 키를 입력하면 된다. 그리고 이 키들에 대한 profile_name을 입력한다. 여기서는 sample-profile-name이라는 임의의 문자열을 입력했다.

## boto3 패키지를 이용해서 S3 버킷 만들기

먼저 pipenv를 이용해 boto3 패키지를 설치한다.

```shell
# pipenv install boto3
```

이어서 아래와 같은 파이썬 코드를 이용하면 된다.

```python
import boto3

# ~/.aws/credentials의 profile_name
session = boto3.Session(profile_name='sample-profile-name')
client = session.client('s3')
bucket_info = client.create_bucket(
    Bucket='bucket-name-sample-1234',
    CreateBucketConfiguration={
        'LocationConstraint': 'ap-northeast-2',
    }
)
```

코드를 읽어보면 boto3 패키지가 ~/.aws/credentials 파일에 접근, 세션을 만들고 세션에서 create_bucket() 메소드를 이용했음을 알 수 있다. 이렇게 코드를 실행하면 bucket_info 변수에 json 타입의 정보를 얻을 수 있다. AWS 콘솔에서 확인해보자.

<div style="text-align: center;"><img src="{{ '/assets/img/s3-5.png' | prepend: site.baseurl }}" alt=""></div>

[boto3 Document]:https://boto3.readthedocs.io/en/latest/
