---
layout: post
title:  "AWS로 Django 프로젝트 배포하기(중급) 3. nginx, uwsgi, supervisor 설정"
date:   2018-7-16
categories: [aws]
---

<p class="intro"><span class="dropcap">D</span>jango 애플리케이션을 배포하기 위해서 웹 서버가 필요하다. 서버와 애플리케이션 사이에는 게이트웨이가 필요하고 서버와 게이트웨이 프로세스를 관리해주는 프로세스 관리자 역시 필요하다. 여기서는 웹 서버로 nginx를 사용하고, wsgi로는 uwsgi, 프로세스 관리자로는 supervisor를 사용할 것이다.</p>

nginx, uwsgi, supervisor는 설정이 필요하다. 다만 주의할 점이 있는데 우리는 EB를 사용하고 플랫폼은 Docker이다. 우리가 생성한 EB에는 자체적으로 nginx를 구동시키고 EB 외부에서 오는 요청을 Docker에게 전달할 것이다. 그렇게 하면 Docker 안에서 EB에서 넘겨받은 요청을 처리할 웹 서버가 필요하게 되는데 우리는 이 Docker 속 nginx의 설정을 다룰 것이다.

uginx, uwsgi, supervisor는 루트 폴더의 ./.config 디렉토리에 저장할 것이다.

## nginx.conf

먼저 nginx.conf의 실행 권한과 daemon 설정을 변경할 것이다. 실행 권한은 Docker 컨테이너 안에서 nginx를 실행할 권한인데 우리는 root를 직접 줄 것이다. 그리고 daemon의 경우에는 supervisor를 통해서 프로세스 관리를 할 것이기 때문에 daemon 설정은 off로 한다.

아래의 nginx.conf 설정파일을 사용하자. 이는 nginx 기본 설정파일에서 처음 2줄만 추가한 것이다.

```nginx
user root;
daemon off;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
```

## nginx_app.conf

nginx 설정과 함께 nginx와 연결할 애플리케이션을 설정해야 한다. 우리는 nginx와 Django를 직접 연결하지 않고 중간에 uwsgi를 통해 요청과 응답을 처리할 것이다. uwsgi와는 소켓 연결을 할 것이다. 그리고 이 설정은 nginx의 site-enabled 디렉토리에 링크로 연결되어 사용될 것이다. 이 연결은 [다음 포스팅]의 Dockerfile에서 처리해야 한다.

그 아래 설정파일을 보자.

```nginx
server {
    listen 7000;
    charset UTF-8;
    server_name *.elasticbeanstalk.com;
    client_max_body_size 128M;

    location / {
        uwsgi_pass  unix:///tmp/app.sock;
        include     uwsgi_params;
    }
}
```

7000은 EB와 이 nginx가 있는 Docker 컨테이너 사이에 7000번 포트를 사용한다는 것을 의미한다. 7000번 포트에 특별한 의미는 없다. 여유가 있는 포트를 사용하면 된다. 그리고 [다음 포스팅]에서 Dockerfile 설정으로 7000번 포트 사용을 입력할 것이다. 그리고 파일 내용 중 `location /`을 통해 모든 URL에 대해서 uwsgi와 /tmp/app.sock 파일을 통해 TCP 통신을 한다는 것을 의미한다.

## uwsgi_socket.ini

nginx와 소켓 통신을 할 uwsgi의 설정 파일이다.

```ini
[uwsgi]
chdir=$(PROJECT_DIR)/app
module=config.wsgi:application
socket=/tmp/app.sock
vaccum=true
logto=/var/log/uwsgi.log
```

chdir은 Django 애플리케이션의 위치이다. 파일을 보면 PROJECT_DIR이라는 이름의 환경 변수가 있는데 이 역시 Dockerfile에서 설정할 것이다. module은 Django에서 생성한 wsgi 인스턴스를 의미하는데 지금 설정은 Django의 기본 인스턴스이다. logto는 uwsgi 로그를 기록하는 경로이며 이 파일은 Docker 컨테이너 안에서 확인할 수 있다.

## supervisor_app.conf

supervisor는 프로세스 관리자로 nginx, uwsgi 서비스 모니터링, 프로세스 중단시 재실행 등을 지원하기 때문에 서버 안정성 유지를 위해서 반드시 이용해야만 한다. supervisor는 설정이 간단하다.

```ini
[program:uwsgi]
command=uwsgi --ini /srv/project/.config/production/uwsgi_socket.ini

[program:nginx]
command=nginx
```

## 설정 파일들을 이용하기

이 설정파일들은 현재 단순히 .config라는 이름의 디렉토리에 저장되어 있는 상태이다. 이 파일들은 Django와는 직접적으로 관련이 없다. 그래서 이 파일들은 EB 안에 있는 Docker 컨테이너 안에서 적절한 역할을 하도록 Docker 컨테이너를 빌드할 때 파일들을 옮겨야 한다. 그런데 Docker 컨테이너는 Docker 이미지를 통해 빌드되기 때문에 Docker 이미지를 생성해야 한다.

한편 awsebcli에서는 자동으로 Dockerfile 파일의 이미지에서 Docker 컨테이너를 만들어낸다. 즉, 이제 우리는 Dockerfile을 만들어야 한다. 이것은 다음 포스팅에서 이어서 진행해보겠다.

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

[다음 포스팅]:https://yeojin-dev.github.io/blog/aws-django-intermediate-4/
