---
layout: post
title: MySQL8 설치하기 (CentOS7)
tags: [mysql, mysql8, 설치, centos, centos7]
---

MySQL 8 설치는 아래의 순서대로 한다.

### 1. RPM 설치

```shell
rpm -ivh https://dev.mysql.com/get/mysql80-community-release-el7-2.noarch.rpm
```

### 2. YUM 설치

```shell
yum install mysql-community-server -y
```

### 3. 서비스 시작

```shell
systemctl start mysqld
```

### 4. 임시암호 확인

서비스를 처음 시작할 때 로그 파일에 임시암호가 남는다. 반드시 서비스를 시작한 후에 아래 명령을 실행하자.

```shell
grep 'temporary password' /var/log/mysqld.log
```

### 5. 루트 암호 변경

mysql_secure_installation 으로 루트 암호를 변경한다. 암호는 대문자와 특수문자가 섞여 있어야 한다.

```sql
mysql_secure_installation
```

### 6. 사용자 추가

```sql
CREATE USER '아이디'@'호스트' IDENTIFIED BY '비밀번호';
CREATE USER '아이디'@'호스트' IDENTIFIED WITH mysql_native_password BY '비밀번호';
```

* 아이디 : 원하는 아이디를 입력한다.
* 호스트 : 아이디로 접속하는 호스트를 입력한다. 로컬에서만 접속한다면 'localhost' 를 입력하고 모든 원격 호스트에서 접속한다면 '%' 를 입력한다. 만약, 특정 원격 호스트에서의 접속만을 허용한다면 아이피를 입력하면 된다.
* 비밀번호 : 대문자와 특수문자가 포함된 비밀번호를 입력한다.
* WITH mysql_native_password : **mysql_native_password** 는 인증 플러그인의 이름이다. MySQL 8 의 기본 인증 플러그인은 **caching_sha2_password** 인데 이전 MySQL 클라이언트 라이브러리는 이 인증 플러그인을 지원하지 않는다. 만약 MySQL 8 에서 오래된 MySQL 클라이언트 라이브러리를 사용한다면 WITH mysql_native_password 를 추가해야 한다.

### 7. 권한 설정

{% highlight sql linenos %}
GRANT ALL PRIVILEGES ON *.* TO '아이디'@'호스트' WITH GRANT OPTION;
FLUSH PRIVILEGES;
{% endhighlight %}

* 아이디와 호스트는 6. 사용자 추가를 참고한다.
* WITH GRANT OPTION : 다른 사용자에게 자신이 소유한 권한을 부여한다.
* \*.\* 는 모든 데이터베이스와 테이블에 권한을 준다는 뜻이다.