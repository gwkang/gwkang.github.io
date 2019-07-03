---
layout: post
title: python3 install
subtitle: CentOS7 에 python3 설치하기
tags: [python, install, python3]
---

## python3 를 CentOS7 에 설치해보자.

python3 를 빌드하기 전에 zlib-devel 과 libffi-devel 을 먼저 설치해야 한다. 설치가 안되어 있으면 빌드에 실패하므로 반드시 설치하자.

```shell
shell> yum -y install zlib-devel libffi-devel
shell> wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz
shell> tar xvf Python-3.7.3.tgz
shell> cd Python-3.7.3; ./configure --enable-optimizations && make altinstall
```

root 계정에서 위의 명령을 순서대로 입력하면 python3 가 설치된다. python3 를 실행하기 위해서는 ./python3.7 을 입력해야 하므로 적당한 이름의 링크를 만들어준다.

```shell
shell> ln -s /usr/local/bin/python3.7 /usr/local/bin/python3
```

