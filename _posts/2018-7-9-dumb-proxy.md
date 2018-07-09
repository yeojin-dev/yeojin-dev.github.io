---
layout: post
title:  "멍청한 프락시"
date:   2018-7-9
categories: [web]
---

<p class="intro"><span class="dropcap">C</span>onnection: Keep-Alive 헤더를 이해할 수 없는 멍청한 프락시(dumb proxy)는 클라이언트와 서버 사이에 문제가 생길 수 있다. 이를 보완하기 위해 Proxy-Connection 헤더가 존재하지만 완벽한 해결책은 아니다.</p>

웹 클라이언트 요청에 Connection: Keep-Alive 헤더[^1]가 있으면, 클라이언트가 현재 연결하고 있는 TCP 커넥션을 끊지 않고 계속 유지하려고 한다. 클라이언트가 웹 서버에 요청을 보내려는 중이라면, 그 요청으로 생긴 커넥션이 keep-alive가 되기를 원한다는 의미로 Connection: Keep-Alive 헤더를 보낸다. 만약 서버가 지원한다면 Connection: Keep-Alive를 응답에 포함할 것이다.

## Dumb Proxy

그런데 만약 프락시가 Connection 헤더를 이해하지 못 하는 멍청한 프락시라면 어떻게 될까?

프락시는 Connection 헤더를 그대로 웹 서버에 전송하고 서버는 응답에 다시 Connection을 실어서 보내준다. 프락시는 이를 그대로 클라이언트에 보낼 것이다.

서버는 프락시로부터 Connection 요청/응답을 주고받았기 때문에 TCP 연결을 끊지 않는다. 여기서 문제가 발생한다.

프락시는 서버가 TCP 연결을 끊을 때까지 계속 대기 중인 상태이다. 이 상황에서 클라이언트가 다시 요청을 보내면 프락시는 하나의 TCP 커넥션에 두 요청이 왔기 때문에 이를 무시한다. 그러면 서버로 2번째 요청이 가지 않은 상태이기 때문에 클라이언트는 계속 무응답 상태의 화면만 보여줄 것이다. 이 상황은 서버가 타임아웃으로 커넥션을 자동으로 끊을 때까지 계속된다.

그림으로 다시 확인해보자.

![멍청한 프락시](https://geekeasier.com/wp-content/uploads/2017/05/httpatomoreillycomsourceoreillyimages96926.png)

## Proxy-Connection

위와 같은 문제를 해결하기 위해서 Proxy-Connection이라는 헤더가 존재한다. 이 헤더는 멍청하지 않은 영리한 프락시를 위한 헤더이다. 만약 영리한 프락시라면 Proxy-Connection 헤더를 Connection 헤더로 변경해 서버로 전달하고, 서버의 Connection 헤더를 받아 그대로 클라이언트에 전달한다. 클라이언트는 Proxy-Connection이라는 헤더를 전송했지만 Connection 헤더가 응답으로 온 것을 통해 프락시가 영리한 프락시임을 알 수 있다.

만약 프락시가 멍청한 프락시라면 Proxy-Connection을 그대로 서버로 보낼 것이고, 서버는 이를 무시할 것이다. 이렇게 되면 위에서 설명한 멍청한 프락시 문제를 해결할 수 있다.

## Smart Proxy와 Dumb Proxy

하지만 Proxy-Connection도 문제가 생길 수 있다. 만약 클라이언트 - 영리한 프락시 - 멍청한 프락시 - 웹 서버로 이어지는 구조라면 어떻게 될까?

그림을 보자.

![영리한 프락시와 멍청한 프락시](https://geekeasier.com/wp-content/uploads/2017/05/httpatomoreillycomsourceoreillyimages96926.png)

그림을 보면 영리한 프락시는 Proxy-Connection을 Connection으로 보내준다. 하지만 다음 프락시가 멍청한 프락시라면 이 Connection 헤더를 웹 서버로 보내게 되고 웹 서버는 Connection 헤더를 멍청한 프락시로 보낼 것이다. 이는 맨 처음과 같은 문제를 다시 일으킬 것이다.

## HTTP/1.1의 지속 커넥션

위와 같은 문제로 HTTP/1.1은 Connection: Keep-Alive를 지원하지 않는다. HTTP/1.1에서는 모든 커넥션을 지속 커넥션으로 관리하되 Connection: Close 헤더를 명시했을 때 커넥션을 끊는다. 클라이언트는 이 Connection: Close 헤더가 없으면 커넥션을 유지하는 것으로 추정[^2]할 것이다.

[^1]:이 헤더는 사용하기로 결정되어 HTTP/1.1에는 빠졌다. 하지만 많은 서버와 클라이언트가 이를 계속 사용하고 있기 때문에 웹 애플리케이션을 개발할 때 주의하여야 한다.
[^2]:그러나 서버가 항상 커넥션을 유지했음을 보장하지는 않는다. 서버는 언제든지 커넥션을 끊을 수 있다.
