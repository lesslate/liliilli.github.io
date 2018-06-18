---
layout: post
title: OPGS16 스크립트(컴포넌트) 기능 구현
tags: 'C++, OPGS16'
categories: OPGS16
comments: false
date: 2018-02-21 14:07:39
---

## 시작

![OPGS16](..\..\..\..\..\images\201802\21\example.png)

OPGS16 작업에 착수하기 전에도 몇 번씩 게임 엔진을 만들어볼까 하고 생각한 적은 었다. 물론 그 때는 C++ 도 잘 몰랐거니와 무리해서 만든다고 쳐도 메모리 누수가 줄줄 새는데다가 그래픽 관련 라이브러리의 지식도 하나도 몰랐기 때문에 잠깐 만들다가 쓰레기통에 던져지기만 했다.

그래도 이번에는 OpenGL 도 방학 도중에 알았겠다, C++ 도 저번 학기에 고생하면서 배운데다가 Effective C++ 를 정독했겠다 싶어 OpenGL 샘플 프로젝트를 만들던 것을 선회시켜 레트로 게임엔진을 만들기로 했었고, 예상외로 순조롭게 진행되었다. 스크립팅을 만나기 전까지는...

## Unity 따라해보기

언리얼도 그렇거니와 유니티 같은 게임 엔진에서는 오브젝트가 있고, 거기에 스크립트를 추가해서 행동을 하도록 하는 구조가 일반적이다. (cocos2d-x는 없었던 걸로 기억한다.)

하지만 어떻게 하면 특정 오브젝트의 특정 스크립트를 정적 캐스팅을 통해서 가져오게 할 수 있을까? 가 문제였는데... 왜냐면 유저에 의해서 스크립트가 무수히 많아질 수도 있고, dynamic_cast<> 을 쓰면 한 스크립트를 가져오기 위해서 엄청나게 많은 시간이 걸릴 수도 있었기 때문이었다. 그렇다고 무턱대고 static_cast<> 을 쓰면 Runtime error가 터진다.

### Unity 에서는?

![Unity](..\..\..\..\..\images\201802\21\unity.png)

유니티에서는 [프리팹](https://docs.unity3d.com/Manual/Prefabs.html) 혹은 씬 안에 인스턴스화된 오브젝트마다 개별의 스크립트를 할당해서 행동을 설정하는게 가능하다. 위의 스크린샷을 보면, *Player* 라는 프리팹에 *Player* 라는 스크립트가 할당된 것을 알 수 있다.

이 스크립트는 오브젝트가 초기화 됬을 때 초기화를 하고, 업데이트 프레임마다 `Update()`을 호출해서 오브젝트가 행동을 할 수 있도록 한다.

다음 코드는 Player 스크립트의 코드 중 일부다.

{% highlight csharp %}
private UiManager uiManager;

/**
  * @brief Binds this to singleton, and set oldTime.
  */
private void Awake() {
    oldTime = Time.time;
    uiManager = GameObject.Find("Canvas").GetComponent<UiManager>();
    RefreshLivesToUi();
    RefreshScore();
}

private void RefreshLivesToUi() {
    if (uiManager != null) {
        Debug.Log("Player lives : " + health);
        uiManager.UpdateLives(health);
    }
}

private void RefreshScore() {
    if (uiManager != null) {
        uiManager.AddScore(0);
    }
}
{% endhighlight %}

여기서 위의 `GameObject.Find("Canvas").GetComponent<UiManager>()` 라는 코드를 볼 수 있고, 거기서 가져온 **무언가**를 `uiManager` 에 할당하는 것을 볼 수 있다. 우선 `Find()` 는 씬 안에서 해당 네임을 가진 오브젝트를 반환한다. 그리고 그 오브젝트의 컴포넌트를 가져오는 `GetComponent<>` 을 사용해서 해당 오브젝트의 특정 스크립트를 가져온다.

즉, *Canvas* 라는 이름을 가진 오브젝트에서 *UiManager* 라는 스크립트를 가져와 참조형으로 반환해서 쓸 수 있도록 하겠다는 의미를 가진다. 이렇게, Unity 는 오브젝트가 가진 스크립트 혹은 여러 가지 컴포넌트를 가져와서 다른 오브젝트에서 쓸 수 있도록 한다. (물론 NullPointerException 은 감수해야 한다만)

예외 혹은 런타임 에러만 잘 관리해주면 상당히 편해보이는 시스템이었기 때문에 이것을 OPGS16 에서도 구현해보고자 했다.

### 구현 전

하지만 어떤 식으로 해야할 지 감이 안잡혔다. 위에서도 말했듯이 모든 컴포넌트를 dynamic_cast<> 로 변환해서 nullptr 인가 아닌가를 검증하는 건 시간 낭비일 뿐만 아니라 성능 저하도 있다. 오로지 **static_cast<>** 만으로 해당 스크립트를 다운캐스팅해야 하는 상황이다. 그렇지만 유저에 의해 만들 수 있는 스크립트 개수는 무수하기 때문에 유연한 방법을 찾을 수가 없었다.

그래서 눈을 돌려본 것이 템플릿 메타 프로그래밍(Template meta programming) 이었다. 나름대로 머리를 굴려서 생각해낸 것이 enum 혹은 using aliasing 을 이용해서 거기에 형인자를 저장한 다음, *SFINAE* 로 타입 형질 검사를 써서 true_type 혹은 false_type 을 반환하게 해서 적절하게 에러없는 static_cast<> 을 유도할 수 있게끔 한다는 것이었다.

하지만 조금 더 생각을 해보니 이 방법은 순엉터리라는 것을 깨달았는데, 왜냐면 스크립트는 컴포넌트 컨테이너에 기본 타입으로 업캐스팅 되서 저장되고 있었고, 타입 검증을 하기 위해서는 어찌됬던 간에 받아오고 싶은 형인자를 virtual 함수에 넘겨줘야 하며 이는 불가능하기 때문이다. (그리고 코드에서 냄새도 날 것 같았다)

하지만 인터넷을 찾아보고 있던 차에 (역시 Stackoverflow에서) 나와 같은 고민을 한 사람이 몇 명 더 있다는 것을 알았으며, 질문글과 답글도 올라와 있는 것을 알게 되었다.
[How does Unity's GetComponent() work?](https://stackoverflow.com/questions/44105058/how-does-unitys-getcomponent-work)

## 구현

답은 Macro 을 이용한 전처리 메타 프로그래밍을 사용하는 것이었다. 처음에는 '이게 무슨...'이라고 생각을 했지만 아무리 생각해도 답이 위 글에서 제시된 대로 만들어보는 것 밖에 없었다.

구현의 가장 큰 비중을 차지하는 요소는 **std::hash** 와 **클래스 static 멤버 변수** 이다. 각 컴포넌트 및 파생된 스크립트의 클래스에 static 멤버 변수를 선언하고, 이를 해당 클래스의 타입을 해쉬화한 값으로 초기화한다. 그리고 GetComponent<> 을 이용해 해당 컴포넌트의 해쉬 값과 참조하고 있는 컴포넌트의 최종 해쉬값을 비교하는 방식으로 스크립트 등을 가져올 수 있게 할 수 있다.

여기서 최종 해쉬값을 비교해야 할 때는 파생된 스크립트 클래스의 static 변수 해쉬값에 접근할 수 있도록 `virtual` 가상 함수를 이용해야 한다. static 변수의 이름은 같아도 된다. 왜냐면 바인딩 되는 범위가 각자 다르기 때문이다. 이 가상 함수 안에서 해쉬값을 비교해서 같으면, 해당 타입으로 다운캐스팅한 컴포넌트의 포인터를 반환한다. (포인터를 반환하는 이유는 컴포넌트를 저장하는 컨테이너가 unique_ptr<> 을 담기 때문이다)

ps : 사실 이렇게 되면 받는 쪽에서 널체킹을 할 수 없기 때문에 shared_ptr<> 로 구현하고, weak_ptr<> 을 반환하게 하면 어떨까 생각해봤지만 막상 성능이 걱정되서 실행에 옮기기 어렵다.

### Add, Get, RemoveComponent<>()

스크립트를 구현한다고는 했는데 기왕인거 스크립트를 컴포넌트 클래스의 파생 클래스로 잡고, 컴포넌트 시스템을 구현하기로 했다. 각 오브젝트가 초기화될 때 같이 따라올 컴포넌트도 같이 초기화되어야 하기 때문에, 각 오브젝트의 생성자 바디에서 AddComponent<>() 템플릿 함수로 컴포넌트를 넣어주는 방식을 선택했다.

#### 1. AddComponent<\_Ty, \_Args...>()

AddComponent<>() 는 컴포넌트 기본 클래스의 파생 클래스 타입과, 파생 클래스 타입 각각의 생성자 인자를 받는다. 벌써부터 [보편 참조(Universal Reference)](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)
와 [완벽 전달(Perfect Forwarding)](https://www.justsoftwaresolutions.co.uk/cplusplus/rvalue_references_and_perfect_forwarding.html)이 따라왔다. (사실 오브젝트 트리를 만든답시고 구현한 Instantiate<Ty, Args...>() 함수에서도 이미 써먹긴 했다.)

{% highlight cpp %}
template<
    class _Ty,
    typename... _Params,
    typename = std::enable_if_t<std::is_base_of_v<component::Component, _Ty>>
} void Object::AddComponent(_Params&&... params) {
    m_components.emplace_back(std::make_unique<_Ty>(std::forward<_Params>(params)...));
}
{% endhighlight %}

*m_components* 는 `unique_ptr<Component>` 타입을 담는 벡터 컨테이너다. 사실 몇일 전까지만 해도 `emplace_back` 대신에 `push_back` 을 썼는데 지금은 전달 효율성을 위해 `emplace_back`을 쓰기로 했다.

여기서는 위에서 말한 hash 값이 아직 쓰이지는 않는다.

#### 2. GetComponent<\_Ty>()

GetComponent<\_Ty>() 는 컴포넌트 클래스의 파생 타입만을 받는다. 그리고 타입을 받아서 그 타입에 대한 해쉬값을 컨테이너의 각 요소의 해쉬값과 비교를 해서 적절한 포인터를 반환한다. 만약 찾고자 하는 타입의 컴포넌트가 없으면 `nullptr` 을 반환한다.

GetComponent 자체의 코드는 다음과 같다.

{% highlight cpp %}
template<
    class _Ty,
    typename = std::enable_if_t<std::is_base_of_v<component::Component, _Ty>>
}   _Ty* const Object::GetComponent() {
    for (auto& item : m_components) {
        if (item->DoesTypeMatch(_Ty::type)) return static_cast<_Ty*>(item.get());
    }
    return nullptr;
}
{% endhighlight %}

여기서 SFINAE 을 이용해서 Component 파생형 클래스만을 받게하는 이유는, 컴포넌트 기본 타입을 상속해야만 위 코드에 보이는 `DoesTypeMatch(hash)` 및 해쉬 값을 써먹을 수 있기 때문이다. (어차피 쓰라고 있는 기법인데 안 쓰면 또 골룸하다)

이제 DoesTypeMatch() 에서는 어떻게 구현되야 하는지 알아봐야 한다. 컴포넌트 기본 클래스는 `virtual` 가상 함수 `DoesTypeMatch(const size_t)` 와, 해쉬값을 저장할 size\_t 형의 변수, type 을 가진다. 기본 클래스의 DoesTypeMatch() 는 그저 해쉬값을 인수로 가져와서 type 과 비교해 같은가 다른가를 넘겨주기만 한다. (type 이 public 으로 지정되어 있는 데, private 으로 바꿔도 아무 지장없다.)

{% highlight cpp %}
class Component {
public:
    Component() = default;
    virtual ~Component() = default;

     virtual void Update() = 0;

    /*!
     * @brief Return true/false flag whether or not your finding class is this.
     * @param[in] type_value Hashed type value of type which you want to find.
     * @return True/False flag, if you found proper class return true else false.
     */
    inline virtual bool DoesTypeMatch(const size_t type_value) const noexcept {
        return type == type_value;
    }

public:
    static const size_t type;
};
{% endhighlight %}

그리고 임의의 파생형 컴포넌트 클래스를 하나 만든다. 본래 목적은 개별 스크립트를 가져오는 것이기 때문에 모든 스크립트의 모태가 되는 기반 컴포넌트를 하나 만들자. 이름은 ScriptFrame 이다.

여기서 매크로를 이용한 <strike>정신나간</strike> 기법을 적용한다.

{% highlight cpp %}
/*! ... */

#define SET_UP_HASH_MEMBERS_DERIVED()                                                   \
public: static const size_t type;

#define OVERRIDE_TYPEMATCH(__BASE__, __DERIVED__)                                       \
public:                                                                                 \
inline virtual bool DoesTypeMatch(const size_t type_val) const noexcept override {      \
    if (__DERIVED__::type == type_val)                                                  \
        return true;                                                                    \
    else return __BASE__::DoesTypeMatch(type_val);                                      \
}

#define SET_UP_TYPE_MEMBER(__BASE__, __DERIVED__)                                       \
SET_UP_HASH_MEMBERS_DERIVED()                                                           \
OVERRIDE_TYPEMATCH(__BASE__, __DERIVED__)
{% endhighlight %}

그리고 스크립트 컴포넌트의 마지막에는 이렇게만 적어놓는다.

{% highlight cpp %}
class ScriptFrame : public component::Component {
    /*! ... */

    /*! Create members related to type hash value. */
SET_UP_TYPE_MEMBER(component::Component, ScriptFrame)
};
{% endhighlight %}

모든 파생 컴포넌트 (및 파생의 파생 컴포넌트) 는 오직 `SET_UP_TYPE_MEMBER(_A_, _B_)` 만을 써서 해쉬값과 오버라이딩을 자동으로 선언할 수 있다. 어찌된 것일까?

우선 위 호출 매크로에 첫번째로 다시 불려지는 매크로는 그냥 해쉬 값을 저장할 타입을 선언하기만 한다. 하지만 두번째는 약간 복잡해보인다. 여기서 알아둘 점은, 매크로의 인자는 쓰임에 따라서 형이 되거나, 문자열이 되거나, 값이 될 수 있다. 이것을 이용해서 자동으로 `DoesTypeMatch` 을 오버라이딩해서 값 매칭을 하고, 잘못되면 계층적으로 위층의 `DoesTypeMatch` 멤버 함수를 불러올 수 있게 한다.
(지금와서 생각이 드는게 가상 함수가 외부에 그대로 노출된 상태라서 잘못 쓰일 수도 있겠다는 생각이 들었다만... 이 주제는 나중으로 미루자.)

그리고 파생한 컴포넌트의 해쉬 값을 초기화해야 한다. 이때도 매크로가 유용하게 쓰인다.

{% highlight cpp %}
#define TO_STRING(__TYPE__) #__TYPE__   /*! Convert arguement to string literal */

#define SET_UP_HASH_VALUE_OF(__TYPE__)                                                      \
const size_t __TYPE__::type = std::hash<std::string>{}(TO_STRING(__TYPE__));

/*! ... */
{% endhighlight %}

{% highlight cpp %}
#include <functional>       /*! std::hash */
#include <string>           /*! std::string */
#include "_macro.h"         /*! Macros */

#include "component.h"                              /*! component::Header file */
#include "script_frame.h"                           /*! component::ScriptFrame */

namespace component {
SET_UP_HASH_VALUE_OF(Component)
SET_UP_HASH_VALUE_OF(ScriptFrame)
/*! ... */
}
/*! ... */
{% endhighlight %}

`_type_hash.h` 파일은 기반 클래스를 포함한 모든 컴포넌트의 해쉬 값을 초기화한다. 다만 해쉬를 초기화 할 때, 네임스페이스는 무시하기 때문에 위와 같이 네임스페이스 안에서 매크로를 불러와야 한다.

이렇게 각 컴포넌트의 설정이 끝나면, 드디어 GetComponent<> 을 초기화시킨 클래스에 대해 쓸 수 있게 된다.

#### 3. RemoveComponent<\_Ty>()

마지막은 컴포넌트를 지운다. 어차피 생성자에서 오브젝트에 쓰일 모든 컴포넌트를 추가하고 오브젝트가 사라질 때까지 평생 쓰기 때문에 실용성은 없지만 (게다가 다른 스크립트로 갈아끼운다고 쳐도 그냥 바꿀 스크립트를 가진 오브젝트로 바꿔치기하면 된다) 그래도 Add 랑 Get 이 있으면 Remove 도 있어야지 생각해서 구현해 보았다.

{% highlight cpp %}
template <
    class _Ty,
    typename = std::enable_if_t<std::is_base_of_v<component::Component, _Ty>>
>
bool Object::RemoveComponent() {
    auto it = std::find_if(m_components.cbegin(), m_components.cend(),
                           [](const std::unique_ptr<component::Component>& item) {
        return item->DoesTypeMatch(_Ty::type);
    });
    if (it != m_components.cend()) {
        m_components.erase(it);
        return true;
    }
    else return false;
}
{% endhighlight %}

RemoveComponent 멤버 함수 템플릿은 제거가 성공했는지 실패했는지를 Bool 값으로 반환한다. 함수 바디 안에서는 find_if 을 이용해서, 제거하고자 하는 컴포넌트 타입과 일치하는 아이템의 이터레이터를 반환한다. 만약 반환된 이터레이터가 유효한 요소라면, 해당 요소를 지우고 true 을 반환하지만 아니라면 false 을 반환할 뿐이다.
(글을 쓰면서 또 생각이 드는 것이, m_components 의 컨테이너 타입을 std::list 로 두면 더 좋을 것 같다는 생각이 든다..)

## 실 응용예

![Application](..\..\..\..\..\images\201802\21\app.gif)

몇일 전에 이 기능을 십분 활용해서 아주 간단한 게임 아닌 게임 을 만들어본 적이 있다. 사각형 물체가 이동하면서 비를 맞으면 스코어가 올라가는 게임이었는데, 비를 맞으면 GameCanvas 라는 오브젝트의 스크립트에 접근해 스코어를 올리고, 비 오브젝트는 사라지는 메커니즘을 가졌다.

{% highlight cpp %}
/*! ... */

void Drops::ScoreUpTrigger() {
    RainUiManager* const i = SceneManager::GetInstance().GetPresentScene()
        ->GetObject("GameCanvas")->GetComponent<RainUiManager>();
    if (i) {
        i->AddScore(256);
        ObjectManager::GetInstance().Destroy(GetObject());
    }
}
{% endhighlight %}

GameCanvas 는 게임 내내 사라질 우려가 없기 때문에, NullPointer 로 인한 에러를 당할 일은 없지만 게임이 복잡해질 경우에는 그럴 가능성도 있어서 이 부분에 대해서는 조금 더 고민을 해봐야 할 것 같다.

## 마치며

* 매크로를 사용한 전처리 메타 프로그래밍의 세계를 잠시 맛봤다.
* TMP + PMP... 존나 좋군?
* Stackoverflow 에 경의를 표함.
* Unity 의 그 기능 혹은 비슷한 기능을 구현하고자 하는 분들께 도움이라도 되었으면.
* <strike>배고파 힘들어 글쓰기 힘들어</strike>
* THE END