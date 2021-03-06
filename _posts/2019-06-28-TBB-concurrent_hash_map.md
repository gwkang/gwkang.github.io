---
layout: post
title: tbb::concurrent_hash_map
subtitle: 멀티스레드에서 빠른 해시맵 사용하기
tags: [tbb, map, concurrent, concurrent_hash_map]
---

## tbb::concurrent_hash_map

### 정의

concurrent_hash_map 은 동시 접근을 허용하는 해시테이블이다. 멀티쓰레드 환경에서 빠른 해시맵이 필요하다면 concurrent_hash_map 이 적당하다. **concurrent_hash_map<Key, T, HashCompare>** 형태로 정의할 수 있다. 컨테이너에 저장되는 원소 타입은 **std::pair<const Key, T>** 이다. 다음은 키와 데이터타입이 모두 int32_t 인 맵을 정의한 예제이다.

{% highlight c++ linenos %}
#include <tbb/concurrent_hash_map.h>
using map_t = tbb::concurrent_hash_map<int32_t, int32_t>;
map_t map;
{% endhighlight %}

### Accessor

컨테이너에 저장된 원소에 접근할 수 있는 스마트 포인터이다. 읽기용과 쓰기용이 구분된다. 읽기용은 **const_accessor** 쓰기용은 **accessor** 이다. 아래는 쓰기용 accessor 를 이용해 데이터를 추가하는 예제이다.

{% highlight c++ linenos %}
map_t::accessor acc;
int32_t key = 3;
int32_t val = 15;
if (map.insert(acc, key))
{
    acc->second = val;
}
{% endhighlight %}

해시맵에 이미 데이터가 있다면 insert 명령은 실패한다.  쓰기용 accessor 는 쓰기락을 건다. 언락은 accessor 가 파괴될 때 자동으로 언락된다. 다음은 데이터 추가에 실패하면 데이터를 삭제하는 예제이다.

{% highlight c++ linenos %}
map_t::accessor acc;
int32_t key = 3;
int32_t val = 15;
if (map.insert(acc, key))
{
    acc->second = val;
}
else
{
    // 쓰기락이 아직 풀리지 않았다.
    map.erase(key); // 데드락!!
}
{% endhighlight %}

이 예제는 제대로 동작하지 않는다. 데이터가 제대로 추가된다면 문제 없지만, 추가에 실패해서 데이터를 삭제할 때 데드락이 걸리게 된다. 왜냐하면, 쓰기락이 풀리지 않은 상태에서 erase 를 호출하는데 erase 내에서도 쓰기락을 걸기 때문이다. 데드락을 피하려면 다음과 같이 해야 한다.

{% highlight c++ linenos %}
map_t::accessor acc;
int32_t key = 3;
int32_t val = 15;
if (map.insert(acc, key))
{
    acc->second = val;
}
else 
{
    acc.release(); // 직접 락을 해제한다.
    map.erase(key);
}
{% endhighlight %}

다음은 읽기용 const_accessor 를 이용해 데이터를 읽는 예제이다.

{% highlight c++ linenos %}
map_t::const_accessor acc;
int32_t key = 3;
int32_t val;
if (map.find(acc, key))
{
    val = acc->second;
}
{% endhighlight %}

const_accessor 는 읽기락을 건다.