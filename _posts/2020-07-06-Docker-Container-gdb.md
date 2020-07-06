---
layout: post
title: Docker Container에서 gdb를 사용하기
tags: [docker, container, gdb]
---
# Docker Container에서 gdb를 사용하기

여기에선 도커 컨테이너 안에서 gdb를 설치해서 사용하는 방법을 설명한다.

## 1. docker run

gdb의 모든 기능을 사용하기 위해 `docker run`명령에 옵션 `--cap-add=SYS_PTRACE --security-opt seccomp=unconfined`를 반드시 추가해야 한다.

```shell
docker run --name myapp -d --cap-add=SYS_PTRACE --security-opt seccomp=unconfined
```

`docker-compose`를 이용한다면 다음과 같이 옵션을 추가한다.

```yaml
cap_add: SYS_PTRACE
security_opt:
  - seccomp:unconfined
```

## 2. docker exec

컨테이너 안으로 들어간다.

```shell
docker exec -it myapp /bin/bash
```

## 3. gdb 설치

컨테이너 안에서 gdb를 설치한다. 컨테이너 안에 잠깐 설치하는 것이므로 컨테이너가 삭제되면 같이 삭제된다. 

```shell
yum install -y gdb
```

## 4. gdb 실행

컨테이너 안에서 gdb를 실행한다. 다음의 명령은 실행중인 프로세스에 attach한다.

```shell
gdb <binary_path> <pid>
```



