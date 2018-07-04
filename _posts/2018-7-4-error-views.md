---
layout: post
title:  "Error Views"
date:   2018-7-4
categories: [django]
---

<p class="intro"><span class="dropcap">D</span>jango Built-in Views 중에는 에러에 대한 화면을 렌더링하는 view가 있다. 이 view를 통해 손쉽게 HTTP 에러에 대응하는 웹페이지를 만들 수 있다. </p>

원문은 여기를 참고하자. Django 버전은 2.0이 기준이다. (이 포스트는 번역 포스트가 아니다. 하지만 포스트의 내용이 많이 들어가 있으며 보다 쉽게 설명하는 식으로 쓰려고 노력했다.)

**[Error views 원문(Django Document)]**

## The 404 (page not found) view

```defaults.page_not_found(request, exception, template_name='404.html')```

404 에러는 클라이언트가 서버와 통신할 수는 있지만 서버가 요청한 바를 찾을 수 없다는 것을 가리키는 HTTP 표준 응답 코드이다. Django에서는 위에 나오는 내장 view - page_not_found()를 통해 자동적으로 에러 페이지를 보여준다. 메소드 패러미터를 보면 알 수 있듯이 template 루트 디렉토리에 404.html 파일을 자동으로 렌더링한다.

**즉, 별도의 설정 없이 template 디렉토리 내에 404.html 파일을 만들어두기만 하면 된다.**

404.html을 렌더링할 때 이 view에서는 다음의 2가지 context를 사용한다

1. **request_path** : 에러를 일으킨 URL
2. **exception** : 예외 메시지

## The 500 (server error) view

```defaults.server_error(request, template_name='500.html')```

500 에러는 서버 측 에러이다. 즉, 여러 이유로 Django의 view에서 에러가 발생할 경우 이 view로 인해 template 디렉토리의 500.html이 렌더링된다. 이 view는 별도의 context가 없다.

## The 403 (server error) view

```defaults.permission_denied(request, exception, template_name='403.html')```

403 에러는 서버에 요청이 도달했으나 서버가 접근을 거부할 때 반환하는 HTTP 응답 코드이다. 위의 두 view와 마찬가지로 이 viewsms 403.html을 렌더링한다. context는 아래와 같다.

1. **exception** : 예외 메시지

특이한 점은 이 view는 PermissionDenied라는 예외가 발생했을 때에도 동작한다. 즉, 어떤 이유로 권한 문제가 생겼을 때 아래와 같이 코드를 작성하면 자연스럽게 403.html을 부를 수 있게 된다. 그리고 자세한 예외 내용은 context에 담긴 expection으로 보여줄 수 있다.

```python
raise PermissionDenied
```

## The 400 (bad request) view

```defaults.bad_request(request, exception, template_name='400.html')```

400 에러는 클라이언트 에러이다. request의 문법이 잘못되었다는 의미로 서버의 에러인 500 에러와 대조되는 에러이다. 이 에러는 Django에서 어떤 문제가 있는지 알기 힘들기 때문에 별도의 context는 없다. 그리고 403 에러와 마찬가지로 SuspiciousOperation라는 예외를 일으켜서 view를 동작하게 할 수 있다.

```python
raise SuspiciousOperation
```

## Debug = False

위의 4개의 view 중에서 404, 500, 400 에러는 settings.py 파일의 Debug = False인 경우에만 동작한다. Debug = True일 경우에는 에러의 자세한 상황을 개발자에게 보여주기 위해 별도의 웹페이지를 만들고 이 때문에 view는 동작하지 않는다.

[Error views 원문(Django Document)]:https://docs.djangoproject.com/en/2.0/ref/views/
