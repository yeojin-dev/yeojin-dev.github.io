---
layout: post
title:  "MVC model"
date:   2018-6-5
categories: [web]
---

<p class="intro"><span class="dropcap">M</span>VC model은 소프트웨어 디자인 패턴의 하나로 많은 웹 프레임워크에서 적용하고 있다.</p>

MVC 모델을 간략히 도식으로 표현하면 아래와 같다.

![MVC](https://upload.wikimedia.org/wikipedia/commons/thumb/5/53/Router-MVC-DB.svg/300px-Router-MVC-DB.svg.png)

## Model

모델(Model)이란 어떠한 동작을 수행하는 코드를 말하며 주로 데이터베이스를 관리해주는 DBMS에 해당된다. 모델은 사용자와 직접 맞닿아있지 않기 때문에 표시 형식에 의존하지 않는다. 컨트롤러가 모델의 상태를 변경, 관리하며 모델을 어떻게 설정하느냐에 따라 해당 웹 애플리케이션이 어떤 정보를 다루는지가 결정된다. 모델은 데이터를 뷰로 보내서 데이터를 사용자가 볼 수 있게 해준다.

## View

뷰(View)는 여러 컨트롤러를 가지고 있고 컨트롤러를 통해 사용자에게 보여줄 데이터를 모델로부터 얻어온다. 그래서 뷰는 가져온 데이터를 사용자에게 전달하기 위한 UI 역할을 한다고 말할 수 있다.

## Controller

컨트롤러(Controller)는 모델에 명령을 보내 모델의 상태를 바꾼다. 뷰가 모델의 데이터를 가져오는 것과는 대조적이다. 사용자는 뷰를 통해서 모델이 제공하는 정보를 볼 수 있는 한편, 컨트롤러를 통해 모델의 상태를 바꿀 수 있다. 모델의 상태가 바뀌면 모델은 등록된 뷰에 자신의 상태가 바뀌었다는 것을 알리고 뷰는 거기에 맞게 사용자에게 모델의 상태를 보여 준다.

## MVC model in Django

장고에서는 MVC model 대신에 MTV model을 사용한다. MTV model이란 Model, Template, View를 의미하는데 이는 각각 Model-Model, View-Template, Controller-View와 일치한다. MVC model과 MTV model에서 View가 서로 다른 개념인 것에 주의하자.
