---
layout: post
title: CentOS 7 에서 cmake 최신버전 설치하기
tags: [centos, cmake, latest]
---
## cmake 최신 버전 내려받기

이 글을 쓰는 시점에 최신 버전은 3.16.0 이므로 이 기준으로 설명한다.

```shell
> wget https://github.com/Kitware/CMake/releases/download/v3.16.0-rc3/cmake-3.16.0-rc3.tar.gz
> tar xvf cmake-3.16.0-rc3.tar.gz
> cd cmake-3.16.0-rc3/
```

## bootstrap 실행 하기

```shell
> ./bootsrap
```

## Make!!

이제 빌드한다.

```shell
> make -j$(nproc) && make install
```

