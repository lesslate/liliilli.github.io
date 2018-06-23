---
layout: post
title: "Reverse OOP 패턴"
tags: 
date: 2018-06-23 +0900
categories: C++
comments: false
---

# Reverse OOP?

{% youtube https://www.youtube.com/watch?v=GnWASmocihE %}

Pope Kim님의 최신 영상을 보던 중에 꽤 흥미가 가는 설계 패턴이 있어서 한번 쭉 보게 되었다. 이 패턴은 다형성을 좀 특이하게 구현해서 런타임에 불필요한 코드를 만드는 것을 막는다. 흔히 동적 다형성을 구현한다고 하면 모든 자식 클래스들의 행동을 공유하는 베이스 클래스가 있고, 그 베이스 클래스를 자식 클래스들이 `public` 과 같은 접근 지정자로 상속을 해서 구현한다. 

하지만 ***Reverse OOP*** 에서는 런타임에서 여러 클래스 중 하나만 쓰되, 코드상으로는 다형성을 구현할 수 있도록 한다는 것이다. 어떻게 보면 정적 다형성이라고도 할 수 있겠다. 왜냐면 다형성이 컴파일 타임에 이뤄지기 때문이다. 엄밀하게 말하면 전처리(preprocessing) 시간에 해당 클래스의 다형성 설계를 끝장내버리는 `C++` 만의 괴랄한 방법이기도 한 것 같다...

## 되는대로 구현

1. 가장 기본이 되는 베이스 클래스를 구현한다.
2. 베이스 클래스에 대해 선언을 한 후, `virtual` 을 대신할 함수의 정의를 각 자식 클래스에 대해 따로따로 `.cpp` 파일에서 구현한다.
3. 역(reverse)자식 클래스가 되는 클래스들을 구현한다. 여기서 데이터들은 `protected` 접근 지정자를 가지도록 한다.
4. 이제 베이스 클래스를 역자식 클래스의 자식이 되도록 역(reverse)상속을 한다. 다만 전처리 매크로를 활용해서 매크로가 활성화된 자식 클래스에 대해서만 상속을 받게끔 한다.
  * 전처리 매크로인 `#define` `#ifdef` `#else if` `#endif` 을 적극적으로 활용해서 전처리 타임 다형성을 구현한다.
5. 베이스 클래스의 자식 클래스에 의존성을 가지는 함수 코드에서 `protected` 로 지정된 역자식 클래스의 데이터에 접근을 할 수 있게 된다.

다음은 예제 코드다. 여기서는 한 파일에만 구현을 했기 때문에 자식 클래스와 의존성을 가지는 `Draw()` 함수는 아주 간단하게만 구현했다.

{% highlight cpp %}
#include <iostream>

///
/// Platform specific implementation
///

// #define MACRO_RENDER_PC
// #define MACRO_RENDER_SWITCH
#define MACRO_RENDER_XBOX
// #define MACRO_RENDER_PS4

class RendererPc {
public:
    RendererPc() { std::cout << "RendererPc()""\n"; }
    virtual ~RendererPc() { std::cout << "~RendererPc()""\n"; }

protected:
    const int32_t i = 0x0001;
};

class RendererSwitch {
public:
    RendererSwitch() { std::cout << "RendererSwitch()""\n"; }
    virtual ~RendererSwitch() { std::cout << "~RendererSwitch()""\n"; }

protected:
    const int32_t i = 0x0010;
};

class RendererXbox {
public:
    RendererXbox() { std::cout << "RendererXbox()""\n"; }
    virtual ~RendererXbox() { std::cout << "~RendererXbox()""\n"; }

protected:
    const int32_t i = 0x0100;
};

class RendererPs4 {
public:
    RendererPs4() { std::cout << "RendererPs4()""\n"; }
    virtual ~RendererPs4() { std::cout << "~RendererPs4()""\n"; }

protected:
    const int32_t i = 0x1000;
};

///
/// RenderCommon implementation.
///

#if defined(MACRO_RENDER_PC)
    #define MACRO_RENDERER_SUPER RendererPc
#elif defined(MACRO_RENDER_SWITCH)
    #define MACRO_RENDERER_SUPER RendererSwitch
#elif defined(MACRO_RENDER_XBOX)
    #define MACRO_RENDERER_SUPER RendererXbox
#elif defined(MACRO_RENDER_PS4)
    #define MACRO_RENDERER_SUPER RendererPs4
#endif

class RendererCommon final : public MACRO_RENDERER_SUPER {
public:
    RendererCommon() : MACRO_RENDERER_SUPER() {};
    ~RendererCommon() = default;

    int32_t Draw() noexcept {
        return i;
    }
};

int main() {
    RendererCommon instance{};
    return instance.Draw();
}
{% endhighlight %}

이렇게 하면 `#define` 을 껐다 키는 것만으로 플랫폼 혹은 다른 정적 다형성이 필요한 부분에서 컴파일 타임에 다른 행동을 할 수 있도록 한다. 일반 동적 다형성으로 구현되는 모델에서는 `vtable` 을 통해서 간접 참조를 2번 이상 해야하기 때문에 성능 면에서도 우위를 가진다.

## `std::variant` 로 그냥 써버리면 안되나?

`C++17` 부터 추가된 `std::variant` 는 템플릿 흑마술을 써서 컴파일 타임 정적 다형성을 구현하게 한 `cpp` 에 맞게 구현된 유니언 타입이다. 한 프로그램에 여러 개의 정적 타입이 들어가게 되면 이걸 써도 되기야 하겠지만, 지금은 오로지 `#define` 을 써서 한번에 한 가지 프로시져를 쓰려고 하고 있기 때문에 필요는 없을 것 같다.