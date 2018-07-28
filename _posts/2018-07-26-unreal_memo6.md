---
layout: post
title: "UE4 메모 (9) Artificial Intelligence"
tags: 
date: 2018-07-26 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## Simple Game Tutorial in UE4

> [https://www.raywenderlich.com/168091/create-simple-game-unreal-engine-4](https://www.raywenderlich.com/168091/create-simple-game-unreal-engine-4)

## AI and Behavior Tree

> [AI and Behavior Tree](https://docs.unrealengine.com/en-us/Gameplay/AI)

> [Behaviour Tree Quick Start Guide](https://docs.unrealengine.com/en-US/Engine/AI/BehaviorTrees/QuickStart)

> [わからないまま使っている？UE4 の AI の基本的なこと](https://www.slideshare.net/rarihoma/ue4-ai-88981638)

이 프로젝트에서는 `UE4` 의 `Behavior Tree` 을 사용해서 `AI` 을 구현한다.

AI 캐릭터를 만들기 위해서는 비유로 3 가지 요소가 필요하다.

하나는 `Body` 인데, 이는 캐릭터의 `Soul` 이랑 `Brain` 이 들어갈 `AActor` 와 같은 것을 말한다. `Soul` 은 `Brain` 에 의해 캐릭터를 컨트롤하는 객체와 같은 것이 된다. `Brain` 은 C++ 코드, BP, 혹은 UE4 에서 제공하는 `Behavior Tree` 을 말한다.

### Controller

`Controller` 는 `Pawn` 을 소유할 수 있는 비물리적인 `AActor` 이다. 즉, 소유한다는 것은 해당 `Pawn` 을 어떻게든 조종할 수 있다는 것을 의미한다. `Pawn` 은 컨트롤러 혹은 행동트리, C++, 블루프린트에서 정보를 받아서 액션을 취할 수 있다.

여기서는 `AI Controller` 을 써서 AI 을 구현해본다.

### AI Controller

### `Behavior Tree` 만들기

`BT` 을 만드려면 `Add New > Artificial Intelligence` 에서 `BT` 을 생성한다. 이름 프리픽스는 `BT_` 가 좋다. Behavior Tree 에디터는 다음과 같다.

![img_behavior]({{ "https://koenig-media.raywenderlich.com/uploads/2017/12/unreal-ai-07-650x352.JPG" | absolute_url }})

1. `Behavior Tree` 는 `Root` 로부터 시작되는 행동 트리를 만든다.
2. `Details` 는 `Behavior Tree` 의 각 행동 노드의 자세한 속성을 보여준다.
3. `Blackboard` 는 `Blackboard keys` 라는 것을 보여주는데, 게임이 런타임으로 돌아갈 때만 활성화된다고 한다.

`Behavior Tree` 는 5 가지 노드로 구성된다. 하나는 `Task` 하나는 `Composite` 이다.

### `Task` 와 `Composite`

> [Behavior Trees Nodes Reference](https://docs.unrealengine.com/en-us/Engine/AI/BehaviorTrees/NodeReference)

* `Task` 는 행동 트리의 잎과 같으며, 출력 연결을 가지지 않고 `APawn` 에게 무언가를 하도록 하는 역할을 한다. 기다리거나, 공격을 하는 등의 행동이 가능하다.
* `Composite` 는 나뭇가지의 루트를 자처하는 역할을 하면서 그 밑에 연결된 또 다른 나뭇가지가 어떻게 연결되는 가를 정의한다. 또한 어떻게 밑의 잎들을 실행할지도 결정한다.

`Task` 을 실행시키기 위해서는 위쪽으로 `Composite` 라는 것이 필요하다. 그리고 이 `Composite` 의 속성을 변경해서 병렬, 시퀀스, 등등으로 사용할 수 있다.

예를 들어서 `APawn` 을 움직이고 `Enemy` 를 향해 회전한 뒤에 공격을 한다고 하면, 다음과 같이 행동 트리를 구성할 수 있다.

![img_unreal_ai]({{ "https://koenig-media.raywenderlich.com/uploads/2017/12/unreal-ai-09.JPG" | absolute_url }})

> [Behavior Tree Node Reference: Composites](https://docs.unrealengine.com/en-us/Engine/AI/BehaviorTrees/NodeReference/Composites)

`Composite` 에 구성되는 일련의 `Task` 시퀀스들 중 하나라도 실패할 경우에는 다음 행동을 하지 않는다. 이것이 원칙이다. 예를 들면 적이 하나라도 없어서 움직일 수가 없다면, 해당 행동뿐만 아니라 적을 향해서 회전한다거나 공격하는 행동은 하지 않게 된다.

`Sequence Composite` 외에도 `Selector Composite` 도 있는데 이는 왼쪽에서 오른쪽으로 행동을 하면서 하나라도 성공한 행동이 있다면 실행을 멈추도록 한다. 어찌보면 일련의 부수 행동을 셀렉터에 달고, 이 중 하나를 실행하게 하는 것과 같다.

#### 임의 방향으로 움직여보기

다음과 같이 행동 트리를 구성한다. 

![img26_bt_moverandomly]({{ "/assets/201807/img26_bt_moverandomly.JPG" | absolute_url }})

여기서 왼쪽에서 오른쪽으로 순서를 지키는 것이 중요하다. 각 `Node` 에 순서가 나와있기 때문에 이걸 잘 이용하자.

* 문제는 `MoveTo` 계열의 `Task` 노드는 오직 `Blackboard` 을 통해서 얻은 값만을 사용해서 특정 월드 좌표로 이동한다. 따라서 `Blackboard` 을 하나 만들어줘야 한다.

> 여기까지 `AI Controller` `Behavior Tree` `Blackboard` 을 만들어 줘야한다.

### Blackboard

> [Blackboard Documentation](https://forums.unrealengine.com/development-discussion/blueprint-visual-scripting/2138-blackboard-documentation)

`Blackboard` 는 단독의 함수를 가지며, 이 함수는 `Keys` 라고 불리우는 변수들을 캐치하기 위해 존재하는 에셋이다. (C++ 로도 만들 수 있는 것 같다) 즉, 이 에셋은 `AI` 의 메모리에 비유할 수 있다.

`Blackboard` 을 사용해서 데이터를 읽거나 쓸 수 있다. `Behavior Tree` 의 거의 모든 노드들은 오직 `Blackboard` 만을 사용해서 데이터를 접근할 수 있기 때문에 매우 중요하다.

이를 만들려면 `Add New > Artificial Intelligence > Blackboard` 을 사용해서 `BB_` 계의 BP 을 만든다.

![img_unreal_bb]({{ "https://koenig-media.raywenderlich.com/uploads/2017/12/unreal-ai-12.JPG" | absolute_url }})

`Blackboard` 에디터는 위와 같다.

1. `Blackboard` 는 데이터가 될 `Keys` 을 보여준다.
2. `Blackboard Details` 는 선택된 `Key` 의 속성을 보여준다.

여기서는 `MoveTo` 하기 위한 위치 값 벡터가 필요하기 때문에 `Key` 로 벡터를 선택하자.

그런데 지금 우리가 원하는 벡터 값은 현재 공간에서 랜덤으로 설정된 벡터 값을 필요로 한다. 이 랜덤 값을 생성하기 위해서 필요한 것이 `Service` 이다.

### Service

`Service` 는 일종의 `Task` 와 비슷하지만, `Task` 가 `APawn` 에 직접 행동을 내리는 것과는 다르게 `Blackboard` 의 데이터를 업데이트 하거나 체크 (접근?) 하기 위해 필요한 것이다.

`Service` 는 개별의 노드로 작용하지 않고, `Task` 혹은 `Composite` 에 붙여서 사용한다.
 
서비스를 사용해서 개별 노드에 붙일려면 다음과 같이 한다.

1. `Behavior Tree` 의 툴바에서 `Add Service` 을 사용해서 `BTSerivce_` 로 시작하는 서비스를 만든다. 이는 `Content Viewer` 에서 볼 수 있으며 이름도 바꿀 수 있다.
2. 만들어진 `Service` 을 노드에 붙이기 위해서는 해당 노드에 오른쪽 클릭을 하고 `Add Service` 로 서비스를 추가한다.

![img26_service]({{ "/assets/201807/img26_service.JPG" | absolute_url }})

이제 `Service` 의 `Event Graph` 에 들어가서 현재 컨트롤되고 있는 `APawn` 의 랜덤 로케이션을 가지고 오면 된다.

여기서는 해당 서비스가 붙어있는 `Node` 가 활성화 될 때 로케이션을 구할 예정이기 때문에 `Event Receive Activation AI` 이라는 노드를 사용한다.

> `Event Receive Actvation` 역시 사용할 수 있지만, 이 경우에는 해당 `BT` 가 붙어있는 `APawn` 의 레퍼런스를 가지고 오지 않는다.

![img26_service_event]({{ "/assets/201807/img26_service_event.JPG" | absolute_url }})

물론 반경을 적절하게 설정해줘야 임의 무작위 포지션을 가지고 올 수 있다.

![img26_navmesh]({{ "/assets/201807/img26_navmesh.JPG" | absolute_url }})

* 주의할 점은, `GetRandomPointInNavigableRadius` 와 같이 `Navigable` 이 붙은 이벤트 노드는 `NavMesh` 라고 부르는 말 그대로 내비게이션이 가능한 공간 안에서만 동작한다. 현재 프로젝트는 `NavMesh` 을 미리 만들었지만 임의 프로젝트를 할 경우에는 `NavMesh` 을 직접 만들어줘야 한다.

* `NavMesh` 을 만들고자 한다면 `Nav Mesh Bounds Volume` 을 사용하자. 메인 `Viewport` 에서는 `P` 을 눌러서 `NavMesh` 을 확인할 수 있다.

> [NavMesh Content Example](http://api.unrealengine.com/KOR/Resources/ContentExamples/NavMesh/)

이제 `Serivce` 을 사용하여 `Blackboard` 에 데이터를 저장하도록 한다. 만들어진 랜덤 포지션을 어떻게 저장할지가 문제긴 하다만... 다음 두 가지 방법이 존재한다.

1. `Make Literal Name` 노드를 사용해서 해당 이름을 사용해 키를 저장할 수 있다. (리플렉션?)
2. `BT` 에 변수 키를 노출시킬 수 있도록 할 수 있다.

여기서는 2번을 쓴다고 한다.

1. 우선 `Service` 의 `Blueprint` 에서 변수를 만든다. 이 때 해당 변수의 타입은 `Blackboard Key Selector` 라고 하는 것으로 하고, 밑의 `Instance Editable` 을 사용해서 해당 블루프린트의 객체에서 `public` 접근 권한으로 값 접근을 할 수 있는 것을 활성화한다.
2. 그 후에 `Service` 의 그래프에서 `Set Blackboard Value as ...` 을 사용해 키에 값을 저장한다.

![img26_service_key]({{ "/assets/201807/img26_service_key.JPG" | absolute_url }})

3. 이제 `Behavior Tree` 의 디테일 뷰에서 `Blackboard` 을 설정한다. 그러면 `TargetLocation` 이라는 변수가 자동으로 생성되어 `Blackboard Key Selector` 로 저장된 `public` 한 데이터가 위쪽 상단 `BT` 에서 사용할 수 있게 된다.

> `AI Controller` 가 `APawn` 에 붙어있고 활성화 되어 있으며, `AI Controller` 에서 특정 `BT` 을 활성화 시키고, `BT` 는 노드를 통해서 행동을 취할 준비가 되어 있으며, 노드는 `Blackboard` 와 `Service` 을 사용하고 있어야 한다.

### AI Perception

> [Perception](https://api.unrealengine.com/INT/BlueprintAPI/AI/Perception/index.html)

> [AI Perception in UE4](https://denisrizov.com/2017/05/08/ai-perception-in-unreal-engine-4-how-to-setup/)

> [【UE4】AI Perception の紹介と使い方](https://qiita.com/Dv7Pavilion/items/741402f4da609ec595a2)

* `AI Perception` 은 `AActor`, `Controller` 등 에 추가할 수 있는 컴포넌트이다. 이걸로 `AActor` 에게 자신만의 시야, 소리 등등의 감각을 부여할 수 있다. `Scene-component` 가 아니기 때문에 별도의 리스트에 추가된다.

여기서는 `AI Controller` 이 각종 `APawn` 을 제어하기 떄문에 `AI Controller` 에 `AI Perception` 을 추가해줘야 한다. 그리고 다른 `APawn` 이 움직이는 것을 인식하도록 해야한다. 따라서 `AI Sight config` 을 추가한다.

![img26_aiperception]({{ "/assets/201807/img26_aiperception.JPG" | absolute_url }})

`AI Sight Config` 은 다음과 같은 옵션을 제공하고 있다.

* `Sight Radius` 는 해당 `APawn` 이 볼 수 있는 최대 거리를 말한다.
* `Lose Sight Radius` 는 `APawn` 이 다른 `APawn` 등을 보았을 때 타겟이 된 `APawn` 이 시야에 없어질 때 까지 어느정도의 거리가 필요한지를 나타낸다.
* `Peripheral Vision Half Angle Degrees` 는 `APawn` 의 반쪽 시야각을 나타낸다.

기본으로 `AI Perception` 은 다른 `Team` 에 설정된 `APawn` 을 감지한다. 그런데 현재 이 프로젝트에서 생성되는 `APawn` 들은 팀이 없고 죄다 `Neutral` 을 유지한다. 그래서 `Detection by Affiliation` 에서 중립 역시 클릭을 해서 중립 캐릭터도 볼 수 있도록 한다.

![img26_perceptionview]({{ "/assets/201807/img26_perceptionview.JPG" | absolute_url }})

* `AI Perception` `Behavior Tree` `AI Controller` 가 적용된 `APawn` 이 어떻게 행동하는 가를 볼려면 런타임 게임 뷰에서 `'` 키를 눌러서 `AI Debug Mode` 에 들어간 후에 넘패드 `1``2``3``4` 키로 확인할 수 있다.

### Enemy Key 만들기

이제 다음은 움직이는 `Neutral` 한 다른 `APawn` 을 인식해서 인식한 `APawn` 가 있는 방향으로 움직이는 것을 할 것이다. 이를 할려면 `Behavior Tree` 에서 인식한 `APawn` 에 대해서 알아야 한다.

인식한 `APawn` 의 레퍼런스 역시 `Blackboard` 에서 참조할 수 있다.

![img26_objref]({{ "https://koenig-media.raywenderlich.com/uploads/2017/12/unreal-ai-30.JPG" | absolute_url }})

그리고 나서 `Behavior Tree` 에 `Enemy` 레퍼런스에 움직일 수 있도록 한다. 이 경우 `Controller` 에서 해당 `Enemy` 을 갱신하기 때문에 별도의 서비스가 필요없다.

이제 `Controller` 에서 `AIPerception` 이 업데이트 되었을 때 현재 바인딩된 `Blackboard` 에서 키를 가지고 와서 키가 `nullptr` 인가를 확인 한 후에 현재 인식된 액터 중 타겟인 것을 하나 가지고 와서 바인딩한다.

![img26_controller]({{ "/assets/201807/img26_controller.JPG" | absolute_url }})

### 별도의 `BTTask` 만들기

> [UBTTaskNode](https://api.unrealengine.com/INT/API/Runtime/AIModule/BehaviorTree/UBTTaskNode/index.html)

`Behavior Tree` 에서 노드를 만들지 않고 `BTTask_` 의 독립된 형태로 `Task` 을 만들 수 있다. 여기서는 `BTTask_BlueprintBase` 을 만든다.

![unreal_ai_35]({{ "https://koenig-media.raywenderlich.com/uploads/2017/12/unreal-ai-35.JPG" | absolute_url }})

그리고 `BTTask_BlueprintBase` 태스크 블루프린트 안에서 다음과 같이 설정을 한다.

![img26_attackflag]({{ "/assets/201807/img26_attackflag.JPG" | absolute_url }})

여기서 중요한 것은, 일반 태스크와는 다르게 직접 생성한 `Task` 는 `Finish Execute` 로 태스크가 어떻게 끝나는 지를 결정해줘야 다음 태스크로 넘어가거나 현재 태스크에서 행동을 끝낼 수 있다.

그 후에 해당 태스크를 행동 트리에서 추가한다.

![img26_bt_semifinal]({{ "/assets/201807/img26_bt_semifinal.JPG" | absolute_url }})

그런데 돌려놓고 테스트를 해보면 분명 `Attack` 시퀀스가 실행되서 `APawn` 이 공격을 바로 해야함에도 불구하고 그렇지 않고 `Roam` 이 먼저 실행되어 보이는 것을 알 수 있다. 

통상적인 `Behavior Tree` 에서는 매 업데이트마다 루트에서 순서대로 행동이 실행되어 `Attack` 후에 `Roam` 이 실행되는 것이 정상이다. 만약 `Enemy` 의 참조가 어떻게 되버리면 `Roam` 으로 바로 넘어갈 것이다.

* 하지만 `UE4` 의 행동 트리는 마지막에 실행된 노드에서 행동 트리를 실행을 해버린다. 따라서 `AI Perception` 은 이 상태라면 다른 `AActor` 들을 인식하지 않고 `Roam` 으로 넘어가 버린다. 게다가 `Roam` 시퀀스에서는 5 초 동안 기다리는 태스크가 있기 때문에 `Attack` 을 바로 할 수 없다.

### Decorator

> [Behavior Tree Node Reference: Decorators](https://docs.unrealengine.com/en-us/Engine/AI/BehaviorTrees/NodeReference/Decorators)

* `Decorator` 는 `Behavior Tree` 에서 분기문을 담당하는 노드로서 `Composite` 및 `Task` 에 서비스처럼 부착될 수 있다. 또한 분기문을 설정해서 연결된 서브트리의 실행을 중단시킬 수도 있다.

예를 들면 `Roam` 시퀀스를 진행하다가 `Enemy` 레퍼런스가 바뀌어서 셋이 된것을 알아차리면 `Decorator` 에서 이를 알려서 `Roam` 시퀀스를 도중에 끊고 `Attack` 으로 넘어가게 한다던가 할 수 있다고 한다. 이런 행동을 주기 위해선 `Blackboard Decorator` 을 사용해서 `Blackboard` 에 설정된 키의 값이 변경이 되었는가 안되었는가를 확인하면 된다. 

현재 `Enemy` 레퍼런스는 AIC_Muffin 에서 업데이트 하고 있으며 `AIPerception` 은 (아마) 매 고정된 프레임마다 업데이트를 해서 블루프린트 로직을 지나가게 하고 있기 때문에 자동으로 `Enemy` 레퍼런스는 업데이트가 될 것이다.

다음과 같이 `BT_Muffin` 을 변경한다. 하는거라고는 `Attack` 컴포지트 시퀀스에 `Blackboard decorator` 을 추가하고, `Detail view` 에서 타겟 키를 적 레퍼런스 참조로 변경하는 것 뿐이다.

하지만 이렇게 해서 완전히 끝난 것은 아니고, 이제 `Decorator` 에서 업데이트가 되었을 때 `Roam` 서브트리의 행동을 중단시키기 위해 `Observer Aborts` 을 사용한다.

### Observer Aborts

> [http://www.prontiatvr.com/learning/devblog/8](http://www.prontiatvr.com/learning/devblog/8)

`Observer Aborts` 는 `blackboard` 의 `key` 가 변경이 될 때 서브트리의 행동을 도중 중단하는 역할을 가진다.

여기서 세 가지로 또 나뉜다.

1. `Self` 는 여기서는 `Enemy` 레퍼런스가 참조를 할 수 없을 때 (invalid) 자기 자신의 서브트리 시퀀스 (여기서는 `Attack`) 을 중단하도록 한다. 이는 `Enemy` 타겟에게 공격을 하기 전에 다른 객체에 의해 죽어서 객체 참조가 무효화 될 때 일어날 것이다.
2. `Lower Priority` 는 `Enemy` 가 바뀌어서 참조가 다시 되면, 낮은 우선순위의 행동 트리를 도중에 중단한다. 행동 트리의 우선순위는 실행 순서에 따라서 자동으로 매겨지는데, 여기서는 `Roam` 이 `Attack` 보다 낮기 때문에 `Roam` 도중에 참조가 다시 변경이 되면 중단이 될 것이다.
3. `Both` 는 `Self` 및 `Lower Priority` 을 둘 다 행한다.

이 셋 중에서 옵션을 설정 시에, 해당 `Decorator` 에 의해 영향을 받는 노드들은 랩핑이 될 것이다.

## Environment Query System (EQS)

위 튜토리얼에서는 다루지는 않았지만, `EQS` 라고 하는 것은 `UE4` 에서 사용할 수 있는 `AI` 시스템 중 하나이며, 주위 환경에 대한 정보를 얻은 다음에 얻은 데이터에 대해 질의를 거쳐서 가장 적합한 데이터를 반환하는 기능이다.

이를 사용해서 가장 가까운 파워업 아이템, 어느 적이 가장 위협적인가, 어느 쪽이 가장 안전한가 등을 파악할 수 있다.