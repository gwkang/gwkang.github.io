---
layout: post
title: Ansible Playbook
tags: [ansible, playbook]
---
# Ansible Playbook

플레이북은 인벤토리에 지정된 호스트에 대한 설정들을 정의한다.

플레이북은 하나 이상의 '플레이'들로 구성된다. 플레이의 목표는 호스트 그룹을 잘 정의된 역할(태스크)에 연결하는 것이다.

플레이북을 여러 '플레이'로 작성해서 여러 머신에 배포할 수 있게 한다. (어떤 스텝을 웹서버 그룹의 모든 머신에서 실행하고 어떤 스텝을 데이터베이스 서버 그룹에 실행하는 등)

## 호스트와 유저

각 플레이는 대상이 될 호스트 머신과 태스크를 실행할 유저를 선택해야 한다.

```yaml
---
- hosts: webservers
  remote_user: root
```

리모트 유저는 태스크마다 정의 할 수 있다.

```yaml
---
- hosts: webservers
  remote_user: root
  tasks:
    - name: test connection
      ping:
      remote_user: yourname
```

다른 유저로 실행하는것도 가능하다.

```yaml
---
- hosts: webservers
  remote_user: yourname
  become: yes
```

특정 태스크에 **become** 키워드를 사용할 수 있다.

```yaml
---
- hosts: webservers
  remote_user: yourname
  tasks:
    - service:
        name: nginx
        state: started
      become: yes
      become_method: sudo
```

루트가 아닌 유저로 될 수 있다.

```yaml
---
- hosts: webservers
  remote_user: yourname
  become: yes
  become_user: postgres
```

su 같은 권한 상승 방법을 이용할 수도 있다.

```yaml
---
- hosts: webservers
  remote_user: yourname
  become: yes
  become_method: su
```

sudo 암호를 명시해야 한다면 ansible-playbook 을 --ask-become-pass 나 -K 와 함께 실행하면 된다.

### 정렬

실행될 호스트의 순서를 제어할 수 있다.

```yaml
- hosts: all
  order: sorted
  gather_facts: False
  tasks:
    - debug:
        var: inventory_hostname
```

order 값은 다음과 같다.

* inventory - 기본값. 인벤토리에 있는 순서대로
* reverse_inventory - 인벤토리의 역순으로
* sorted - 이름을 알파벳 순으로
* reverse_sorted - 알파벳의 역순으로
* shuffle - 랜덤하게

## 태스크 목록

플레이는 태스크 목록을 갖고 있다.  태스크는 호스트 패턴에 일치하는 모든 머신에서 한 번에 하나씩 순서대로 실행된다.

태스크는 모듈을 실행한다. 변수는 모듈의 인자로 사용될 수 있다.

모듈은 멱등성이어야 한다. 모듈이 여러번 실행되더라도 결과는 단 한 번만 실행한 것과 같아야 한다. 멱등성을 지원하는 한 가지 방법은 모듈이 최종 상태에 있는지 확인 해서, 최종 상태라면 아무 것도 하지 않는 것이다. 플레이북이 사용하는 모든 모듈이 멱등성을 지원하면, 그 플레이북은 멱등성을 지원한다.

```yaml
tasks:
  - name: make sure apache is running
    service:
      name: httpd
      state: started
```

**command** 와 **shell** 모듈은 리턴 코드를 다룰 수 있다.

```yaml
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand || /bin/true
```

```yaml
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand
    ignore_errors: True
```

## 핸들러: 변화에 실행되는 작업

모듈은 원격 시스템에 변경이 있을 때 전달할 수 있다. 플레이북은 이런 변화에 응답할 수 있는 기본 이벤트 시스템을 갖고 있다.

'알림'은 플레이에 있는 각 태스크 블럭의 끝에서 트리거된다. 그리고 여러 다른 태스크에서 알림이 발생하더라도 한 번만 트리거된다.

예를 들어, '여러' 리소스가 설정 파일을 변경해서 아파치를 재시작할 필요가 있게 할 수도 있다. 그러나 아파치는 불필요한 재시작을 피하기 위해서 (설정파일이 여러번 바뀌더라도) 한 번만 재실행 될 것이다.

아래 예제는 파일의 내용이 바뀌면 두 서비스를 재시작한다.

```yaml
- name: template configuration file
  template:
    src: template.j2
    dest: /etc/foo.conf
  notify:
     - restart memcached
     - restart apache
```

notify 섹션에 나열된 것들을 핸들러라고 부른다.

* 핸들러는 태스크의 목록이다.
* 이것은 실제로 보통의 태스크와 다르지 않다.
* 핸들러에게 어떤 알림도 가지 않는다면, 태스크들은 실행되지 않는다.
* 여러 태스크가 핸들러에게 알림을 보내더라도, 특정 플레이의 모든 태스크가 완료된 후에 한 번만 실행된다.

```yaml
handlers:
    - name: restart memcached
      service:
        name: memcached
        state: restarted
    - name: restart apache
      service:
        name: apache
        state: restarted
```

