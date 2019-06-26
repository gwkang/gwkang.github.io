---
layout: post
title: brew cask install java
subtitle: Homebrew로 Java 설치하기
tags: [brew, homebrew, java]
---

Mac 에서 Homebrew 로 자바를 설치하기! 우선 cask 를 이용해야 한다. 아래 명령은 최신 버전의 자바를 설치한다.

```shell
shell> brew cask install java
```

글을 작성하는 시점에는 버전 12가 설치됐다. 만약 버전 8을 설치하고 싶다면 아래와 같이 해야한다.

```shell
shell> brew tap adoptopenjdk/openjdk
shell> brew cask install adoptopenjdk/openjdk/adoptopenjdk8
```

설치 후에 버전을 확인해보자

```shell
shell> java -version
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_212-b03)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.212-b03, mixed mode)
```



* [https://dev.mysql.com/downloads/connector/cpp/](https://dev.mysql.com/downloads/connector/cpp/)
