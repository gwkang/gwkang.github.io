---
layout: post
title: Ansible Zabbix
tags: [ansible, zabbix, docker]
---
# Ansible 로 Zabbix 를 설치하기

* ansible 을 이용해서 zabbix-server 와 zabbix-agent 를 설치해보자.
* 설치할 서비스는 모두 docker 를 이용한다. 우선 ansible 을 설치한다.
* ansible 의 [role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) 을 이용한다.

```shell
> yum install ansible -y
```

## 설치할 항목

* mysql server
* zabbix server
* zabbix agent

mysql server 와 zabbix server 는 같은 호스트에 설치한다.

## hosts

우선 호스트 목록을 지정한다. 파일은 작업 폴더 루트에 위치하고 이름은 hosts 이다.

```ini
[server]
server01	ansible_host=1.1.1.1

[mysql]
mysql01		ansible_host=1.1.1.1

[agent]
agent01		ansible_host=2.2.2.2
agent02		ansible_host=3.3.3.3
agent03		ansible_host=4.4.4.4
```

zabbix server 와 mysql server 는 같은 호스트(1.1.1.1)에 설치한다. zabbix agent 는 각 호스트에 설치한다.

## 암호문 만들기

아래의 명령으로 암호문이 저장돼 있는 .password 파일을 만든다. 이 파일은 --vault-password-file 옵션의 인자로 넘어가게 된다.

```shell
> openssl rand -base64 1024 > .password
```

## 폴더 만들기

작업 폴더 아래에 다음의 구조로 폴더를 만든다.

* .password
* hosts
* site.yml
* ansible.cfg
* **group_vars/** : 그룹 변수를 저장하는 폴더
  * all : 모든 호스트가 사용할 그룹 변수들이 정의돼 있다.
  * mysql : mysql 호스트들이 사용할 그룹 변수들이 정의돼 있다.
* **roles/**
  * **common/**
    * **tasks/**
      * main.yml
    * **vars/**
  * **mysql/**
    * **handlers/**
    * **tasks/**
      * main.yml
    * **templates/**
  * **server/**
    * **tasks/**
      * main.yml
  * **agent/**
    * **tasks/**
      * main.yml

## SSH 인증 정보 설정

ssh 인증 정보는 노출되면 안되기 때문에 ansible-vault 명령을 이용해서 암호화 해야 한다. 

모든 호스트의 ssh 인증 정보가 같다면, group_vars 폴더 아래에 all 이라는 파일에 ssh 인증정보를 저장하면 된다. 그렇지 않다면 인증 정보가 같은 호스트끼리 그루핑해서 인증 정보를 그룹별로 저장한다. 여기에서는 모든 호스트의 ssh 인증 정보가 같다고 가정한다.

아래의 명령을 실행하고,

```shell
> ansible-vault create group_vars/all --vault-password-file .password
```

아래의 내용을 입력한 후에 저장한다.

```yaml
---
ansible_ssh_user: username
ansible_ssh_pass: password
```

암호화된 내용은 아래의 명령으로 볼 수 있다.

```shell
ansible-vault view group_vars/all --vault-password-file .password
```

group_vars 폴더 아래의 all 이란 이름을 가진 파일은 모든 호스트에게 적용되는 변수가 저장된다.

## mysql 암호 설정

mysql 암호는 그룹변수에 저장한다.

mysql 암호도 ssh 인증 정보와 마찬가지로 노출되면 안되기 때문에, ansible-vault 명령을 이용해서 암호화해야 한다.

아래의 명령을 실행하고,

```shell
> ansible-vault create group_vars/mysql --vault-password-file .password
```

아래의 내용을 입력한 후에 저장한다.

```yaml
---
rootpass: password
dbuser: zabbix
upassword: password
```

## 플레이북 만들기

### site.yml

이 파일은 롤과 호스트를 연결해준다.

```Kyaml
---
- name: apply common configuration to all nodes
  hosts: all
  remote_user: root
  roles:
    - common

- name: install zabbix-server
  hosts:
    - server
  remote_user: root
  roles:
    - server

- name: install zabbix-agent
  hosts:
    - agent
  remote_user: root
  roles:
    - agent

- name: install and start mysql
  hosts:
    - mysql
  remote_user: root
  roles:
    - mysql

```

### roles/common/tasks/main.yml

```yaml
---
- name: install docker
  yum:
    name: docker
    state: latest
```

### roles/mysql/tasks/main.yml

```yaml
---
- name: open firewall
  firewalld:
    port: '{{ item }}/tcp'
    state: enabled
  with_items:
    - 3306
    - 33060
- name: run mysql server
  docker_container:
    name: mysql
    image: mysql
    state: started
    recreate: "yes"
    restart_policy: always
    networks:
      - name: "mysql-server"
    volumes:
      - "/root/volume/mysql/conf.d:/etc/mysql/conf.d"
      - "/root/volume/mysql/data:/var/lib/mysql"
    ports:
      - "3306:3306"
      - "33060:33060"
    env:
      - MYSQL_ROOT_PASSWORD: "{{ rootpass }}"
      - MYSQL_USER: "{{ dbuser }}"
      - MYSQL_PASSWORD: "{{ upassword }}"
    
```

* networks : zabbix-server 와 mysql 을 같은 호스트에 설치하기 때문에, 네트워크 설정을 해야한다. 설정한 네트워크 이름을 zabbix-server 도커 설정에서 사용한다.
* conf.d 파일은 템플릿을 이용한다.

### roles/server/tasks/main.yml

```yaml
---
- name: open firewall
  firewalld:
    port: '{{ item }}/tcp'
    state: enabled
  with_items:
    - 10051
    - 80
- name: run agent server
  docker_container:
    name: zabbx-server
    image: zabbix-server-mysql
    state: started
    recreate: "yes"
    restart_policy: always
    env:
      - DB_SERVER_HOST: "mysql-server"
      - DB_SERVER_PORT: "3306"
      - MYSQL_USER: "{{ dbuser }}"
      - MYSQL_PASSWORD: "{{ upassword }}"
      - MYSQL_DATABASE: "zabbix"
    
```

* duser & upassword : 앞에서 설정한 group_vars/mysql 에 정의 돼 있는 변수다.
* DB_SERVER_HOST : mysql 도커에서 설정한 networks 이름을 설정한다.

### roles/agent/tasks/main.yml

```yaml
---
- name: open firewall
  firewalld:
    port: '{{ item }}/tcp'
    state: enabled
  with_items:
    - 10050
- name: run zabbix agent
  docker_container:
    name: zabbx-agent
    image: zabbix-agent
    state: started
    recreate: "yes"
    restart_policy: always
    privileged: yes
    env:
      - ZBX_HOSTNAME: "{{ inventory_hostname }}"
      - ZBX_SERVER_HOST: "{{ hostvars['server01']['ansible_host'] }}"
```

* privileged : 도커 컨테이너는 기본적으로 unprivileged 이기 때문에 호스트 자원을 엑세스 할 수 없다. 호스트의 시스템을 모니터링하기 위해서는 privileged 모드로 실행돼야 한다.
* ZBX_HOSTNAME : zabbix-server 에서 호스트 설정에 사용하는 이름이다. 인벤토리의 호스트이름을 그대로 사용한다.
* ZBX_SERVER_HOST: zabbix-server 의 주소를 입력한다. hostvars\['server01']['ansible_host'] 를 이용해서 얻을 수 있다.

