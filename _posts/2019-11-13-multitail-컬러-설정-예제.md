---
layout: post
title: multitail 색 설정 예제
tags: [multitail, color, example]
---
## multitail 설치하기

```shell
yum groupinstall -y "Development Tools"
yum install -y wget
yum install -y ncurses-devel

wget https://www.vanheusden.com/multitail/multitail-6.5.0.tgz
tar xvf multitail-6.5.0.tgz
cd multitail-6.5.0
make install

cp /usr/local/etc/multitail.conf.new /usr/local/etc/multitail.conf
```

## multitail 커스텀 색 설정하기

multitail 은 기본적으로 여러 컬러스킴을 지원한다. 하지만 자신이 만든 로그를 보기에는 그 스킴들이 쓸모 없을 수가 있다. 이럴 때 자신만의 컬러 스킴을 만들어 써야 한다.

컬러스킴을 만드는 것은 어렵지 않다. 정규 표현식만 알면 쉽게 할 수 있다.

우선 multitail 을 설치해 보자. 여기에서는 CentOS 7 을 기준으로 설명한다.

```shell
sudo yum install -y multitail
```

그리고 나서 설정 파일 (**/usr/local/etc/multitail.conf**) 에 아래와 같이 내용을 추가한다.

```
colorscheme:example
cs_re:magenta,,bold:^.*\[critical\].*
cs_re:red,,bold:^.*\[error\].*
cs_re:yellow:^.*\[warning\].*
cs_re:green:^.*\[info\].*
```

이렇게 하면 설정은 끝난 것이다. 실행은 다음과 같이 multitail 을 실행하면 된다.

```shell
multitail -cS example -f <logfile>
```

