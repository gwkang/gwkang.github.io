---
layout: post
title: Ansible Inventory
tags: [ansible, inventory]
---
# Ansible Inventory

인벤토리에는 Ansible 이 동작할 시스템 목록을 표시한다. 인벤토리에 대해서는 딱 세 가지만 알면 된다.

1. 그룹
2. 변수
3. 계층

## 호스트와 그룹

인벤토리 파일은 갖고 있는 플러그인에 따라 여러 포맷 중 하나로 만들 수 있다. 예를 들어 기본 인벤토리 파일인 /etc/ansible/hosts 는 INI-like 로 되어 있다.

아래와 같이 단순히 인벤토리에 시스템(호스트)을 나열할 수 있다.

```ini
one.example.com
two.example.com
three.example.com
```

하지만 실제론 아래처럼 역할에 따라 그루핑을 하게된다.

```ini
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

이렇게 그루핑을 하는 이유는 one.example.com, two.example.com, three.example.com 서버에 데이터베이스를 설치하라는 것 보다 dbservers 그룹에 데이터베이스를 설치하라는 것이 더 편리하기 때문이다.

그리고 이렇게 그루핑을 하면 그룹에 변수를 지정할 수도 있기 때문에 편리하다. 

## 변수

### 호스트 변수

인벤토리에 있는 각 호스트마다 변수를 정의할 수 있다. 이렇게 정의한 변수는 플레이북에서 사용하게 된다.

```ini
[webservers]
foo.example.com  myvar=10  yourvar=20
bar.example.com
```

호스트 변수는 각 호스트마다 갖고 있는 고유의 값을 지정하기 위해 정의한다.

### 그룹 변수

그룹 내에 있는 호스트들이 공유하게 되는 변수를 그룹 변수라고 한다. 

```ini
[webservers:vars]
mygroupvar1=10
mygroupvar2=20
```

위와 같이 정의하면 foo.example.com 과 bar.example.com 는 위 두 변수를 갖게 된다. 

## 계층 구조

:children 을 사용해서 그룹의 그룹을 만들 수 있다. 이것도 그룹이기 때문에 그룹 변수를 가질 수 있다.

```ini
[servers:children]
webservers
dbservers

[servers:vars]
svrvar1=10
svrvar2=20
```

