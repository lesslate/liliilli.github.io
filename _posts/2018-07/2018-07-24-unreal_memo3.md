---
layout: single
title: "UE4 메모 (6) 애니메이션 등"
tags: 
date: 2018-07-24 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## Simple Game Tutorial in UE4

> [https://www.raywenderlich.com/168091/create-simple-game-unreal-engine-4](https://www.raywenderlich.com/168091/create-simple-game-unreal-engine-4)

### `Split Struct Pin`

![img_split]({{ "https://koenig-media.raywenderlich.com/uploads/2017/08/3-1.gif" | absolute_url }})

`Blueprint` 에서 벡터 값은 오른쪽 클릭을 통해서 나오는 `Split struct pin` 을 사용해서 $$ (x, y, z) $$ 값으로 변경할 수 있다. 3개의 $$ (x, y, z) $$ 을 리컴바인 하는 경우도 있지만 이 경우에는 링크가 되어있지 않을 경우에만 `Recombine Struct Pin` 을 사용할 수 있다.

### 블루프린트에서 `AActor` 계열 `Spawn` 하는 법

![img_spawn]({{ "/assets/201807/img24_spawn.PNG" | absolute_url }})

`Spawn Actor` 계의 것을 쓰면 된다. 이 때 레퍼런스를 줄 수도 있고 아니면 직접 `AActor` 파생 타입을 지정해서 만들 수도 있다.

또한 `Transform` 은 `World Location`, `Rotation`, `Scale` 로 `Split` 하거나 `Recombine` 할 수도 있다.

### `Is Valid` BP 노드 사용하기

`Is Valid` 에는 두 가지 종류가 있다.

![img_isvalid]({{ "/assets/201807/img24_isvalid.PNG" | absolute_url }})

왼쪽의 것은 조건식을 대신하며 유효한가 아닌가에 따라 `bool` 형태의 `true` 와 `false` 을 반환한다.

오른쪽의 노드는 해당 `Input Object` 레퍼런스가 Null 인가 아닌가에 따라 조건분기를 나누는 노드이다.

### BP 에서 IF 사용하기

`IF` 대신에 `Branch` 라는 노드를 사용해서 조건분기를 한다. 이 때 조건식은 항상 `bool` 의 값을 출력하는 조건식 블루프린트 트리를 사용해야 한다. (`Get` 노드도 당연)

### `Sequence` 노드

![img_sequence]({{ "https://koenig-media.raywenderlich.com/uploads/2017/08/65.jpg" | absolute_url }})

시퀀스 노드는 노드가 일렬로 길어지는 것을 방지하는 블루프린트 노드다. `Then 0` 에 연결된 노드가 끝나면, `Then 1` 에 연결된 노드 트리가 시작되는 형식이다.

### `UMG` 의 위젯 이벤트 처리를 핸들링하기

![img_umgevent]({{ "/assets/201807/img24_umg_event.PNG" | absolute_url }})

`UMG` 디자이너 탭에서 위젯을 클릭하면 다음과 같이 `Details` 에 이벤트 표시가 뜨는 것을 확인할 수 있다. 이벤트 옆의 `+` 을 클릭해서 블루프린트 노드를 구성할 수 있다.

![img_eventimpl]({{ "/assets/201807/img24_event_impl.jpg" | absolute_url }})

### `USceneComponent`

> A SceneComponent has a transform and supports attachment, but has no rendering or collision capabilities. Useful as a 'dummy' component in the hierarchy to offset others.



### 그 외

> `LocalRotation` 등의 회전 각도의 단위는 `Degree(도)` 이다.

## UE4 Animation Tutorial

> [https://www.raywenderlich.com/171595/unreal-engine-4-animation-tutorial](https://www.raywenderlich.com/171595/unreal-engine-4-animation-tutorial)

### Importing a Skeletal Mesh

이 예제에서는 `.fbx` 을 사용한다. Bone 스켈레털 애니메이션을 포함하는 확장자는 여러가지가 있지만, `.dae` 라던가 `.fbx` 가 많이 쓰인다고 한다.

![img_fbx]({{ "/assets/201807/img25_fbx.png" | absolute_url }})

`.fbx` 파일의 메쉬를 불러오면 `UE4` 에서는 기본적인 설정창으로 설정을 하도록 한다. `Create Physics Asset` 은 각 본에 물리엔진을 적용하도록 하는 플래그인데 지금은 이걸 쓸 일이 없으니 끄도록 하자.

또한 여러 메쉬 파일에 따라서 내부에 머터리얼을 가지고 있는 경우도 있는데, 이 경우 `Material` 에서 머터리얼과 텍스쳐를 따로 사용하는 것을 끌 수 있도록 해야한다.

![img_result]({{ "/assets/201807/img24_result.jpg" | absolute_url }})

하지만 이 상태로는 아직 메쉬에 머터리얼이 적용되지 않았기 때문에 메쉬를 더블 클릭해서 디테일 뷰로 들어간 뒤에 `Material` 에서 적절한 내장 머터리얼들을 선택해 줘야 한다.

그리고 플레이어 Pawn 의 메쉬와 머터리얼을 변경한다. 그러면 다음과 같이 `.fbx` 메쉬가 적용되서 Viewport 에 들어오는 것을 알 수 있다.

![img_goingon]({{ "https://koenig-media.raywenderlich.com/uploads/2017/09/04.gif" | absolute_url }})

하지만 아직 애니메이션은 적용되질 않았다. 왜냐면 지금 불러온 메쉬는 애니메이션이 없는 메쉬이기 때문이다.

### Importing Animations

여기서만 그런 것인지는 모르겠지만, `.fbx` 등과 같은 애니메이션이 담긴 메쉬를 불러올 때에는 기본 베이스가 되는 메쉬가 존재하고, 애니메이션이 담긴 메쉬들은 그 기본 베이스가 되는 메쉬의 스켈레톤을 참조하는 형식으로 사용하는 것 같다.

그러니까 애니메이션이 존재하는 메쉬와 기본 베이스의 메쉬와 파일이 각각 다르다는 것이다.

![img_import]({{ "https://koenig-media.raywenderlich.com/uploads/2017/09/13.jpg" | absolute_url }})

이제 애니메이션 메쉬 파일들을 불러올 때는 `.fbx` 의 경우 `Import Mesh` 플래그를 꺼서 Skeletal Mesh 가 다시 불러오지 않도록 한다. 밑의 스켈레톤은 `Content Viewer` 에 이미 임포트된 스켈레톤이며 해당 애니메이션을 위 스켈레톤을 사용해서 구현을 하겠다는 의미이다.

![img_result2]({{ "/assets/201807/img24_result2.JPG" | absolute_url }})

이제 Animation 이 갖춰졌으니 Animation Blueprint 을 사용한다.

### Creating an Animation Blueprint

> [https://docs.unrealengine.com/en-us/Engine/Animation/AnimBlueprints](https://docs.unrealengine.com/en-us/Engine/Animation/AnimBlueprints)

`Animation Blueprint` 는 일반 블루프린트와 비슷하지만 애니메이션을 동작시키는 데 있어서 그래프 형식으로 수월하게 그래프를 작성할 수 있도록 도와준다.

`ABP (Animation Blueprint)` 을 작성하기 위해서는 `Content Viewer` 에서 `Animation/Animation Blueprint` 을 클릭해서 `ABP` 을 만들어낸다.

![img_abp]({{ "https://docs.unrealengine.com/portals/0/images/Engine/Animation/AnimBlueprints/Interface/AnimGraphUI.png" | absolute_url }})

3. `Graph`
  `Graph` 패널은 두 가지 `Event Graph` 와 `Anim Graph` 라는 그래프 타입을 포함한다. `Event Graph` 는 `Anim Graph` 와 스켈레털 메쉬 포즈를 업데이트 하는 트리거 (상태 변동) 에 사용되는 애니메이션 이벤트 노드를 말한다.
  `Anim Graph` 는 스켈레털 메쉬에 현재 프레임에 대해 적용되는 마지막 포즈를 평가하는데 쓰인다.
  이 두 트리거는 상호작용으로 섞여서 구현될 수 있다.

4. `Details Panel`
5. `My Blueprint`
6. `Anim Preview Editor / Asset Browser`

`Graph` 에서 `ABP` 는 `State Machine` 을 구현해서 애니메이션을 동작시킨다.

#### `State Machine` 을 `ABP` 에서 만드는 법

1. `My Blueprint` 에서 `Anim Graph` 을 선택 한 뒤, `Graph` 에서 오른쪽 클릭으로 `State Machine` 을 새로 생성한다.

2. 상태 머신과 `Final Animation Pose` 노드를 연결한다. `Final Animation Pose` 노드는 애니메이션의 프레임이 최종으로 출력되는 노드이며 각 프레임마다 출력된 동작이 실제로 보이게 된다.

3. `State Machine` 에 진입하여 `Entry |>` 에 연결될 `상태 노드`를 새로 만든다. 여기서는 `Idle` 이라고 하자. 그리고 연결을 시킨다. 그러면 `Idle` 은 `Entry` 시점에서 기본으로 동작하는 상태가 된다.

4. `Idle` 상태 노드를 다시 들어가면 `Asset Browser` 에서 애니메이션 등을 가져다가 해당 `Idle` 노드의 `Final Animation Pose` 와 연결시켜 프레임의 출력을 할 수 있다.

따라서 다음과 같이 연결해서 `Idle` 애니메이션을 출력할 수 있게 되었다.

![img_idle]({{ "/assets/201807/img24_idle.gif" | absolute_url }})

하지만 아직 `ABP` 가 `BP` 에 적용된 것은 아니다. 이제 `ABP` 을 해당 플레이어 폰 `BP` 에 설정을 해줘야 한다.

> 4 번에서, 상태를 만들고 그 안에 실제 애니메이션을 링크하는 방법 외에도 `State Machine` 안의 `Event Graph` 에서 바로 애니메이션을 링크해서 상태화 시키는 것도 가능하다.

#### Animation Blueprint 쓰기

1. 애니메이션을 적용할 `BP` 의 설정 창에 (여기서는 `BP_Muffin`) 에 들어가서, `Mesh` 을 클릭한 후 `Details Viewer` 의 `Animation` 에서 다음과 같이 설정한다.

![img_animation]({{ "https://koenig-media.raywenderlich.com/uploads/2017/09/27.jpg" | absolute_url }})

그러면 최종으로 애니메이션이 적용이 되며 실제 게임 상에서도 움직이는 것을 확인할 수 있다고 한다.

### `Animation` 상태 머신에 상태 추가하기

1. 현재는 애니메이션을 여러개 엮을 필요가 없기 때문에 상태 머신으로 돌아와서 애니메이션을 그냥 끌어당겨서 상태 노드화 시킨다.

2. 그리고 각 상태와 상태를 링크해서 `State` 와 `Conduits` 을 만든다. 그러면 이 도관 (Conduits) 에 자동으로 `Transition Rules` 을 설정할 수 있는 자리가 마련된다. (`Entry` 에서 첫 상태로 진입할 떄는 `Rules` 이 없다)

### Transition Rules

> [Transition Rules](https://docs.unrealengine.com/en-us/Engine/Animation/StateMachines/TransitionRules)

각 `Transition Rules` 은 마지막에 로직을 통해서 상태가 변해야 하는가 아닌가를 확인해야 하기 때문에 항상 `Result` 노드의 반환은 `bool` 이다.

### 플레이어가 점프하는가 떨어지는가를 안에서 알아보기

`ABP` 에서도 `My Blueprint` 을 사용해서 변수, 함수 등등을 사용할 수 있다. 

1. `IsJumping` `IsFalling` `bool` 변수를 만든다.

2. `Event Graph` 에서 `Event Blueprint Update Animation` 에 대해 다음과 같이 노드를 설정한다. 위 노드는 `Event Tick` 과 비슷한 역할을 가진다. `ABP` 이벤트 그래프 안에서는 현재 `ABP` 을 가진 메쉬의 `Pawn` 소유주에 대한 참조를 가지고 올 수도 있다.

![img_eventgraph]({{ "/assets/201807/img24_eventgraph.jpg" | absolute_url }})

3. 그리고 나서 `Transition Rules` 을 클릭해, `Update` 하면서 갱신된 변수들을 사용해 갱신한다. 갱신을 다 하고 난 뒤에는 컴파일을 해서 게임 뷰에 반영이 될 수 있도록 한다.

> 반영 사항을 게임 뷰에서 봐야한다고 하면 `ABP` 의 `Viewport` 와 `Anim Preview Editor` 에서 값을 조작함으로써 애니메이션 트랜지션을 확인할 수 있다.

### Blend Space

> [Blend Space](http://api.unrealengine.com/KOR/Engine/Animation/Blendspaces/)

`Blend Space` 는 `Anim Graph` 에서 샘플링할 수 있는 특수 애셋이며, **최대 두 입력 값에 따라 애니메이션을 블렌딩** 시켜주는 기능이다. 이 기능을 사용해서 걷기, 뛰기, 빨리 뛰기와 같은 세부적인 상태에 대해 하드코딩으로 `State` 와 `Conduits` 을 붙여서 급격하게 복잡도가 높아지는 것을 막을 수 있다.

![img_blendspace]({{ "http://api.unrealengine.com/images/Engine/Animation/Blendspaces/Overview/BlendSpaceDirection.jpg" | absolute_url }})

현재 튜토리얼에서는 플레이어의 `Speed` 을 사용해서 애니메이션 블렌딩을 실시한다.

#### Blend Space 만들고 사용하기

* `Content Viewer > Animation > Blend Space` 혹은 `> Blend Space 1D` 을 사용해서 `BS_` 의 블렌드 스페이스를 생성한다.

그리고 `Asset Details` 에서 `Axis Setting` 으로 블렌딩 변수의 값의 이름, 범위치 등을 설정할 수 있다.

이제 `Blend Space` 설정이 다 끝났으면, 다시 `ABP` 로 돌아가서 상태 머신의 걷기 동작을 `BlendSpace` 애니메이션으로 바꾼다. 하지만 블렌드 값을 변경할 값인 `Speed` 가 0 으로 고정되어 있기 때문에 `Event Graph` 에서 이를 담당하는 변수에 값을 세팅하고, 그리고 그 값을 `Anim Graph` 에서 `BS_` 에 넘겨줘야 한다.

#### `Blend Pose` 을 사용해 죽음 애니메이션 처리하기

> [https://docs.unrealengine.com/en-US/Engine/Animation/NodeReference/Blend#blendposesbyint](https://docs.unrealengine.com/en-US/Engine/Animation/NodeReference/Blend#blendposesbyint)

일반 `State` 와 `Conduits` 을 사용해서 죽음 애니메이션을 처리한다면 금새 상태 머신이 복잡해진다.

![img_death]({{ "/assets/201807/img24_death1.jpg" | absolute_url }})

이럴 때는 `Animation Blueprint` 의 `Blend Poses by XYZ` 을 사용하면 된다. 이것은 `Blend Space` 와 비슷한데 따로 `BS_` 을 만들 필요가 없이 노드 상으로 블렌딩을 해주는 노드이다. 

`Anim Graph` 에서 다음과 같이 설정한다.

![img_blendpose]({{ "/assets/201807/img24_anim.jpg" | absolute_url }})

실제로 플레이를 하면 애니메이션이 스무스하게 움직이는 것을 확인할 수 있다.