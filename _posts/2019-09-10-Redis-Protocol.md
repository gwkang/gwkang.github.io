---
layout: post
title: Redis Protocol
tags: [redis, protocol]
---
## Redis Protocol

* RESP (REdis Serialization Protocol) : Redis 클라이언트가 Redis 서버와 통시하기 위해 사용하는 프로토콜의 이름이다.
* RESP 는 다음의 특성을 갖고 있다.
  * 구현이 단순하다.
  * 파싱이 빠르다.
  * 사람이 읽을 수 있다.
* RESP 는 숫자, 문자열, 배열 같은 다른 타입들을 직렬화 할 수 있다. (에러 타입도 포함)

### Networking layer

클라이언트는 Redis 서버에 포트 6379 에 TCP 연결을 만든다.

### RESP protocol

타입 별 시작 문자

| 타입        | 시작 문자 | 예제                             |
| ----------- | --------- | -------------------------------- |
| 단순 문자열 | +         | +OK\r\n                          |
| 에러        | -         | -Error message\r\n               |
| 숫자        | :         | :1000\r\n                        |
| 벌크 문자열 | $         | $6\r\nfoobar\r\n                 |
| 배열        | *         | *2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n |



## 참고

* https://redis.io/topics/protocol

