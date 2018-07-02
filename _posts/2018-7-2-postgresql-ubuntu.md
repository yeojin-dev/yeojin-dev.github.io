---
layout: post
title:  "Ubuntu 환경에서 PostgresSQL 설치 후 리모트 접속하기"
date:   2018-7-2
categories: [django]
---

<p class="intro"><span class="dropcap">P</span>ostgreSQL은 RDBMS의 하나로서 오픈소스이다. Django에서 사용하는 데이터베이스로서 PostgreSQL을 Ubuntu에 설치했는데 예상치 못 한 문제에 부딪혔다.</p>

#### 일러두기
이 문서는 ibara1454님의 문서 [UbuntuでPostgreSQLをインストールからリモートアクセスまでの手順](https://qiita.com/ibara1454/items/40ce2d82926f48cf02bc)를 많이 참고하였다. 모든 부분을 번역하지는 않았지만 90% 정도는 위 링크의 내용임을 일러둔다.


## role "root" does not exist

AWS를 이용하기 위해서 EC2 - Ubuntu 리눅스에 PostgreSQL을 설치하고 관리 셸인 psql을 실행시켰더니 아래와 같은 에러 메시지가 나왔다.

```shell
$ psql
psql: FATAL:  role "root" does not exist
```

어이가 없었던 점은 슈퍼유저 계정(root)으로 이를 실행해도 같은 문제가 발생한다는 점이다. 메시지에는 'root'라는 이름의 **role이 없다**고 하는데 이것은 무슨 소리일까?

## role

PostgreSQL은 권한 관리에 있어서 role이라는 개념을 사용한다. role은 유닉스의 그룹과 닮았는데 각각의 DB에 접속 또는 수정 권한에 대한 설정값을 가지고 있다.

이 role을 만들기 위해서는 2가지 방법이 있다.

1. 유저 'postgres'로 PostgreSQL에 로그인해서 role을 추가하는 것이다.

2. 셸에서 ```createuser``` 명령어를 사용하는 것이다. 이 명령어는 PostgreSQL 설치와 동시에 설정되는 명령어로서 ```createuser name```을 입력하는 것으로 name이라는 이름의 role을 만들 수 있다.

여기서는 1번째 방법을 사용해보려고 한다.

PostgreSQL을 설치했다면 ```postgres```라는 유저 및 같은 이름의 role이 만들어진다. 기본값으로서 ```postgres``` role은 PostgreSQL의 슈퍼유저이면서 role 변경 권한을 가지고 있다.

그러므로 아래와 같이 계정을 바꾸고 PostgreSQL에 로그인할 수 있다.

```shell
$ su - postgres
$ psql
psql (10.4)
Type "help" for help.
postgres=#
```

유저 이름을 사용하지 않고 psql을 명령어를 사용하면 현재 시스템 유저와 같은 이름으로 PostgreSQL을 사용할 수 있다. 즉, 위 셸 명령어는 아래와 같은 명령어를 의미한다.

```shell
psql --username=postgres --dbname=postgres
```

postgres라는 이름의 계정은 PostgreSQL의 슈퍼유저이기 때문에 PostgreSQL에 접속할 수 있을 것이다. psql에서 ```\du``` 또는 ```SELECT rolname FROM pg_roles;``` 명령어로 현재 role의 리스트를 파악할 수 있다. 현재는 아무런 role로 만들지 않았기 때문에 기본값인 상황이다.

```
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of 
-----------+------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication | {}
```

다른 role을 만들지 않고 postgres 계정으로 데이터베이스를 이용하는 것도 가능하겠지만 슈퍼유저로만 데이터베이스를 이용하는 것은 보안에 문제가 생길 수 있다. 슈퍼유저가 아닌 유저 계정으로 데이터베이스를 이용하기 위해서 role을 만드는 것이 훨씬 좋다.

## CREATE ROLE

새롭게 role을 만들기 위해서는 ```CREATE ROLE name;```[^1] 또는 ```CREATE USER name;``` 명령어를 사용하면 된다. 만들고 나서 role을 수정해도 상관없지만[^2] 만들 때 아래 키워드를 입력하는 것도 가능하다.

- LOGIN
- SUPERUSER
- CREATEDB
- CREATEROLE
- REPLICATION
- PASSWORD

속성의 자세한 정보는 [여기](https://www.postgresql.org/docs/current/static/role-attributes.html)를 참고하자.

그럼 'hellosql'이라는 이름의 role을 만들어보자.

```shell
postgres=# CREATE ROLE hellopsql LOGIN CREATEDB PASSWORD 'hello';
CREATE ROLE
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of 
-----------+------------------------------------------------+-----------
 hellopsql | Create DB                                      | {}
 postgres  | Superuser, Create role, Create DB, Replication | {}
```

이제 role을 만들었으니 아래 2가지 방법 중 하나로 데이터베이스를 만들 수 있다.

1. 셸 : ```sudo -u postgres createdb hellosql --owner=hellosql```[^3]
2. psql : ```CREATE DATABASE hellopsql OWNER hellopsql;```

postgres 계정으로 데이터베이스 목록을 확인해보면 잘 만들어졌음을 알 수 있다.

참고로 psql 접속은 ```psql -u (DB 이름) -U (role 이름)```으로 가능하다.

```shell
postgres=# CREATE DATABASE hellopsql OWNER hellopsql;
CREATE DATABASE
postgres=# \l
                              List of databases
   Name    |   Owner   | Encoding  | Collate | Ctype |   Access privileges   
-----------+-----------+-----------+---------+-------+-----------------------
 hellopsql | hellopsql | SQL_ASCII | C       | C     | 
 postgres  | postgres  | SQL_ASCII | C       | C     | 
 template0 | postgres  | SQL_ASCII | C       | C     | =c/postgres          +
           |           |           |         |       | postgres=CTc/postgres
 template1 | postgres  | SQL_ASCII | C       | C     | =c/postgres          +
           |           |           |         |       | postgres=CTc/postgres
(4 rows)
```

[^1]:삭제는 ```DROP ROLE``` 명령어이다.
[^2]:PostgreSQL의 role 관리 테이블의 이름은 pg_roles이다. 자세한 내용은 [여기](https://www.postgresql.org/docs/8.3/static/view-pg-roles.html)를 참고하자.
[^3]:만약 postgres 이외의 슈퍼유저가 있다면 그 계정으로 만들어도 상관없다.