---
layout: post
title: Deduction Guides
tags: [c++, template, deduction, deduction guides]
---

## Deduction Guides

### C++17 에서의 타입 추론

C++17 에서 새로운 방식의 타입 추론이 생겼다. (1) 템플릿 파라미터를 생성자의 매개변수에서 추론하거나, (2) 함수 표기식의 타입 변환에서 추론하는 것이다.

{% highlight c++ linenos %}
template<typename T1, typename T2, typename T3 = T2>
class C
{
  public:
  // constructor for 0, 1, 2, or 3 arguments:
  C (T1 x = T1{}, T2 y = T2{}, T3 z = T3{});
  …
};
C c1(22, 44.3, "hi");  // OK in C++17: T1 is int, T2 is double, T3 is char const*
C c2(22, 44.3);        // OK in C++17: T1 is int, T2 and T3 are double
C c3("hi", "guy");     // OK in C++17: T1, T2, and T3 are char const*
C c4;                  // ERROR: T1 and T2 are undefined
C c5("hi");            // ERROR: T2 is undefined
{% endhighlight %}

모든 매개변수가 추론 절차에 따라 결정되거나 기본 인자로부터 결정된다. 매개변수 중 일부만 명시하고 나머지들을 추론하는 것은 불가능하다.

{% highlight c++ linenos %}
C<string> c10("hi","my", 42);     // ERROR: only T1 explicitly specified, T2 not deduced
C<> c11(22, 44.3, 42);            // ERROR: neither T1 nor T2 explicitly specified
C<string,string> c12("hi","my");  // OK: T1 and T2 are deduced, T3 has default
{% endhighlight %}

### Deduction Guides

{% highlight c++ linenos %}
template<typename T>
class S {
  private:
    T a;
  public:
    S(T b) : a(b) {
    }
};
template<typename T> S(T) -> S<T>;  // deduction guide
S x{12};         // OK since C++17, same as: S<int> x{12};
S y(12);         // OK since C++17, same as: S<int> y(12);
auto z = S{12};  // OK since C++17, same as: auto z = S<int>{12};
{% endhighlight %}

위 코드에서 템플릿 같아 보이는 생성자를 살펴보자. 이것은 deduction guide 라고 불린다. 이것은 함수 템플릿 같아 보이지만 문법적으로 약간 다르다.

선언 S x(12) 에서 S 를 플레이스홀더 클래스 타입이라 부른다. 플레이스홀더가 사용되면 반드시 변수 이름과 초기화가 뒤따라야 한다. 아래와 같은 문법은 허용되지 않는다.

{% highlight c++ linenos %}
S* p = &x;
{% endhighlight %}

앞의 예에 쓰인 가이드로, S x(12); 는 오버로드 셋으로서 클래스 S 의 deduction guides 에 의해 변수 타입이 추론된다. 

앞의 예에서 deduction guides 와 생성자 S(T b) 사이에는 안보이는 연결이 있다. 그러나 그런 연결이 꼭 필요하지는 않다. 이는 deduction guides 가 어그리게이트 클래스 템플릿과 사용될 수 있다는 뜻이다.

{% highlight c++ linenos %}
template<typename T>
struct A
{
  T val;
};
template<typename T> A(T) -> A<T>;  // deduction guide
{% endhighlight %}

deduction guides 가 없다면 C++17 이더라도 아래와 같이 템플릿 인자를 명시적으로 기입해야 한다.

{% highlight c++ linenos %}
A<int> a1{42};      // OK
A<int> a2(42);      // ERROR: not aggregate initialization
A<int> a3 = {42};   // OK
A a4 = 42;          // ERROR: can’t deduce type
{% endhighlight %}

그러나 deduction guides 가 있다면 아래와 같이 쓸 수 있다.

{% highlight c++ linenos %}
A a4 = { 42 }; // OK
{% endhighlight %}

### 암묵적인 추론 가이드

많은 경우에 클래스 템플릿에 있는 모든 생성자에 추론 가이드가 있는 것이 좋다. 그것은 클래스 템플릿 인자 추론의 설계자가 추론을 위한 암묵적인 메커니즘을 포함하게 한다. 기본 클래스 템플릿의 모든 생성자와 생성자 템플릿에 대해 다음과 같은 암묵적인 추론 가이드를 도입하는 것과 같다.

* 클래스 템플릿의 템플릿 매개변수로 구성된 암묵적인 가이드에 대한 템플릿 매개변수 목록

