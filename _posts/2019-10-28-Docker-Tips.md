---
layout: post
title: Docker Tips
tags: [docker, tips]
---
## 사용하지 않는 이미지 모두 지우기

도커를 이리저리 사용하다 보면 더 이상 사용하지 않는 이미지가 쌓이게 되어 있다. 이것들을 일일이 지우려면 너무 귀찮은데 다음의 명령으로 한 번에 지울 수 있다.

```shell
docker image prune -a
```



