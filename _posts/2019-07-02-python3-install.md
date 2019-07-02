---
layout: post
title: python3 install
subtitle: CentOS7 에 python3 설치하기
tags: [python, install, python3]
---

## python3 를 CentOS7 에 설치해보자.

```shell
shell> yum -y install zlib-devel libffi-devel
shell> wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz
shell> tar xvf Python-3.7.3.tgz
shell> cd Python-3.7.3; ./configure --enable-optimizations && make altinstall
shell> ln -s /usr/local/bin/python3.7 /usr/local/bin/python3
```

root 계정에서 위의 명령을 순서대로 입력하면 python3 가 설치된다.