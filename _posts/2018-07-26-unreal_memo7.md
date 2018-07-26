---
layout: post
title: "UE4 헷갈리기 쉬운 것들 메모."
tags: 
date: 2018-07-27 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## Add Movement Input Node

> [https://api.unrealengine.com/INT/BlueprintAPI/Pawn/Input/AddMovementInput/index.html](https://api.unrealengine.com/INT/BlueprintAPI/Pawn/Input/AddMovementInput/index.html)

> [https://api.unrealengine.com/INT/API/Runtime/Engine/GameFramework/APawn/AddMovementInput/index.html](https://api.unrealengine.com/INT/API/Runtime/Engine/GameFramework/APawn/AddMovementInput/index.html)

![img27_addmovementinput]({{ "/assets/201807/img27_addmovementinput.png" | absolute_url }})

은 해당 노드가 있는 `Event Graph` 가 `APawn` 인지 아니면 `ACharacter` 인지 `APawnBase` 인지에 따라서 동작하는 것이 다르다.

### `APawn` 일때...

자동으로 `World Direction` 을 적용하지 않는다. 다만 안쪽에서 `Movement Input Vector` 에 저장을 해서, `Consume Movement Input Vector` 로 이동량이 있는 벡터를 정하는 것은 가능한 것 같다.

그래서 이 경우에는 반드시 `Consume Movement Input Vector` 후에 `AddActorWorldOffset` 을 사용해서 최종 값을 입력해서 액터가 이동을 하게끔 해야한다.

### `ACharacter` 와 `APawnBase` 일 때

이 경우에는 `Add Movement Input` 에 넣는 족족 움직임이 반영된다고 한다.

---

## Yaw & Pitch & Roll

`UE4` 에서는...

* Roll 은 X 축, Pitch 은 Y 축, Yaw 는 Z 축이다.

이 때 Pitch 축은 시계 방향으로 가는 것이 `+` 방향이며 반대 방향이 `-` 방향임을 주의한다. 이 회전축 개념은 $$ (x, y, z) $$ 다 적용이 되지만 뭔가 헷갈려서...

밑은 마우스의 $$ (x, y) $$ 축 움직임에 따라 카메라 시점이 변하는 것을 나타낸다. 이 때 주의할 점은 미리 만들어진 액터가 아닌 자기 자신이 직접 `APawn` `ACharacter` 등을 사용해서 거기에 카메라를 얹을 때 카메라의 회전을 해당 부모의 액터의 회전과 맞춰야 한다.

![img27_lookat]({{ "/assets/201807/img27_lookat.jpg" | absolute_url }})

![img27_control]({{ "/assets/201807/img27_control.jpg" | absolute_url }})

왜냐면 카메라는 단독의 컨트롤러 (`APawn` 을 소유할 수 있는 `AActor` 로, 소유한 `APawn` 에게 행동을 내리는 것이 가능) 를 가지고 있는데, 이 컨트롤러와 액터의 컨트롤러와 다르기 때문에 액터가 회전을 한다고 한들 카메라 역시 제대로 회전한다거나 상호작용 한다는 보장이 없기 때문이다.

---

## Editable Variables & Private

![img27_editable]({{ "/assets/201807/img27_editable.jpg" | absolute_url }})

블루프린트에서 변수를 생성하거나 쓰다보면 `My Blueprint` 밑의 변수의 옆에 눈 표시가 되어있는 것을 확인할 수 있다. 이는 해당 변수의 `Detail Viewer` 에서 설정할 수 있는 `Private` 와는 다르고, `Instance Editable` 와 역할을 동등히 한다.

클릭을 해서 눈을 뜨게하면 해당 블루프린트 객체 뿐만 아니라, 다른 객체에서도 해당 블루프린트의 변수의 값을 변경할 수 있도록 한다. 그렇지 않으면 변경하는 것은 오직 해당 블루프린트 안에서만 가능하다.

`Private` 는 다들 알겠지만... 자식 블루프린트 클래스에서 접근을 할 수 없도록 하는 접근 지정자이다.

---

## Expose on Spawn

이건 문서 보다가 꽤 재미있는 기능인 것 같아서 적어본다..

블루프린트에서 `Spawn Actor from class` 와 같은, `Spawn` 계열의 노드를 사용해서 `AActor` `UObject` 등을 생성할 수 있는데, 여기서 해당 객체에 `Expose on Spawn` 이 프로퍼티든 옵션이든 적용된 변수가 있으면 다른 블루프린트에서 해당 블루프린트를 `Spawn` 할 때 해당 변수에 값을 설정하면서 객체를 생성하는 것이 가능하다.

![img27_spawn0]({{ "https://docs.unrealengine.com/portals/0/images/Engine/Blueprints/UserGuide/Variables/HT26.png" | absolute_url }})

![img27_spawn1]({{ "https://docs.unrealengine.com/portals/0/images/Engine/Blueprints/UserGuide/Variables/HT27.png" | absolute_url }})

---

## Game View 에 일반 문구를 표시하게 하려면?

`Print String` 을 쓴다. 끝.

---

## Latent Actions

> [UE4 ブループリントでコルーチン的な事をやってみる](http://unrealengine.hatenablog.com/entry/2015/04/24/223000)

... Perform a latent action with a delay (specified in seconds). Calling again while it is counting down will be ignored. 

`Unity` 에서는 `Coroutine` 등으로 로직에 딜레이를 주거나 하는 것이 있는데, `UE4` 에서는 `Latent Actions` 라는 것을 활용해서 딜레이 등등의 지연된 연산을 수행할 수 있다고 한다. 

![img27_delay]({{ "/assets/201807/img27_delay.png" | absolute_url }})

이런 `Latent Actions` 중에서 가장 많이 쓰이는 것이 `Delay` 일 것이다. `Delay` 액션은 내부에서 정한 만큼의 타이머를 설정해서 해당 타이머가 $$ 0 $$ 이 되면 다시 액션을 수행하도록 한다. 

그리고 블루프린트에서 `Latent Actions` 은 우상단에 시계와 같은 아이콘이 붙여져 있으니 확인하기도 쉽다.

> [Delay function equivalent in C++](https://answers.unrealengine.com/questions/340146/delay-function-equivalent-in-c.html)

블루프린트에서는 간단하게 `Delay` 노드를 쓸 수 있지만, `C++` 코드 안에서 작업을 해야할 경우에는 `SetTimer` 와 `FTimerHandle` 을 쓰자.

> [First class continuations in UE-4 Blueprints](http://unktomi.github.io/Latent-Actions-Cont/Cont.html)

---

## Latent Function 만들어보기?

> [https://forums.unrealengine.com/development-discussion/c-gameplay-programming/46342-any-way-to-catch-a-latent-function-being-completed?75086-Any-way-to-catch-a-latent-function-being-completed=](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/46342-any-way-to-catch-a-latent-function-being-completed?75086-Any-way-to-catch-a-latent-function-being-completed=)

