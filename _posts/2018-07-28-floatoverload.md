---
layout: post
title: "부동소수점 오버로딩 체크하는 법"
tags: 
date: 2018-07-28 +0900
categories: C++
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

![img28_ieee754]({{ "/assets/201807/img28_ieee754.jpg" | absolute_url }})

여태까지 모르고 있었는데, 이제와서라도 알게 되었으니 적어본다.

일반 정수 `Integer` 계열의 경우 오버로딩 체크를 다음과 같이 할 수 있다.

{% highlight cpp %}
bool Add(int32_t a, int32_t b, int32_t* res) {
    *res = a + b;

    bool is_overloaded = ((*res - a) != b);
    return is_overloaded;
}
{% endhighlight %}

하지만 `Float` 의 경우에는 정수형 처럼 소수점 오차가 아예 없는 것도 아니거니와, `Nan` `-NaN` 과 같은 특정 형태도 존재하기 때문에 위 방법으로는 오버플로우 체킹을 하기가 힘들다.

그런데 `C++11` 에서는 부동소수점 타입 변수의 값에 대해서 연산을 했을 때, 예외 (C++ 예외는 아니고 플래그를 사용한 예외)가 발생했는가에 따라 아주 간단하게 오버플로우가 되었는가를 확인할 수 있다고 한다.

## 방법

1. 헤더파일 `<cfenv>` 을 사용한다.
2. `std::feclearexcept(FE_ALL_EXCEPT)` 을 사용해서 부동소수점 플래그를 초기화시킨다.
3. 부동소수점 연산을 수행한다.
4. `std::fetestexcept()` 와 [링크](https://en.cppreference.com/w/cpp/numeric/fenv/FE_exceptions) 에 나와있는 매크로 플래그를 사용해서 플로팅 연산에서 어떤 예외가 발생되었는 가를 인자로 넣어 호출한다. 

다음과 같이 수행할 수 있다.

{% highlight cpp %}
#include <iostream>
#include <cfenv>
#include <limits>

int main()
{
    std::feclearexcept(FE_ALL_EXCEPT);
    std::cout << "FLOAT_MAX * 3 = " << 
            std::numeric_limits<float>::max() * 3.0f << '\n';
    if(std::fetestexcept(FE_OVERFLOW)) {
        std::cout << "overflow reported\n";
    } else {
        std::cout << "overflow not reported\n";
    }
}
{% endhighlight %}

