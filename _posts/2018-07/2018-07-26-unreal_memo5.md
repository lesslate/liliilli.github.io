---
layout: single
title: "UE4 메모 (8) Cascade 파티클 시스템 등"
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

### `Cascade` Particle system

> [https://docs.unrealengine.com/en-us/Engine/Rendering/ParticleSystems](https://docs.unrealengine.com/en-us/Engine/Rendering/ParticleSystems)

> [Particle System Reference](https://docs.unrealengine.com/en-us/Engine/Rendering/ParticleSystems/Reference)

`UE4` 는 매우 강하고 견고한 파티클 시스템을 가지고 있다. 이 파티클 시스템을 사용해서 연기, 스파크, 불 등을 표현할 수 있게 된다.

`UE4` 에서 사용하는 파티클 시스템을 `Cascade` 라고 한다. 파티클이라 하는 것은 다른 말로 `Point sprite` 라고 할 수 있는데, 왜냐면 파티클 한 점 한 점이 점에 텍스쳐를 씌워서 렌더링하기 때문이다.

`UE4` 의 파티클 시스템은 `Emitter` 라고 하는 컴포넌트로 구성된다. 이 `Emitter` 컴포넌트에서 파티클을 생성한다. 그리고 `Emitter` 는 `Module` 이라고 하는 컴포넌트를 자식으로 가진다. `Module` 은 해당 이미터에서 방출되는 파티클의 특성들을 가진다. 예를 들면 해당 파티클의 적용되는 머터리얼과 파티클의 속도를 말한다.

![img_particle1]({{ "https://koenig-media.raywenderlich.com/uploads/2017/11/unreal-particles-04.gif" | absolute_url }})

### 파티클 만들어보기

파티클을 만들려면 `Add New` 에서 `Particle System` 을 클릭해서 만든다. 만들어진 파티클을 더블클릭으로 열면 `Cascade` 에디터 창이 열린다.

![img_particle2]({{ "https://koenig-media.raywenderlich.com/uploads/2017/11/unreal-particles-01-650x352.JPG" | absolute_url }})

1. `Viewport` 는 파티클들이 `Emitter` 에서 어떻게 방출되는 가를 보여준다.
2. `Details` 는 `Emitter`, `Module` 등의 각종 속성을 보여준다.
3. `Emitters` 는 현재 파티클의 이미터를 보여준다. 이미터는 또 해당 이미터의 모듈을 리스트로 보여준다.
4. `Curve Editor` 는 해당 이미터의 모듈의 값을 조정할 수 있도록 한다. 하지만 모든 모듈이 커브를 지원하는 것은 또 아니다.

![img_cascade_emitter]({{ "/assets/201807/img26_cascade_emitter.JPG" | absolute_url }})

`Required` 모듈은 파티클 머터리얼과 `Emitter` 길이? 의 필수적인 속성을 정의하는데 쓰인다. 따라서 모든 이미터들은 하나의 `Required` 모듈을 가진다.

만들어진 파티클들은 동적이나 정적으로나 `AActor` 에 붙여서 쓸 수 있다. 다만 적절한 위치와 적절한 로컬 각도를 조절해서 사용해야 한다.

#### `Size By Life` 모듈

`Size By Life` 모듈은 해당 `Emitter` 에서 방출된 각각의 파티클의 생명주기에 따라서 사이즈가 달라지게 할 수 있다. 다만 막 생성한 참으로는 고정 수치로 되어있기 때문에 이제 `Curve` 을 사용해서 조절해야 한다.

`Size By Life` 의 디테일은 다음과 같다.

![img_cascade_sizebylife]({{ "/assets/201807/img26_cascade_sizebylife.JPG" | absolute_url }})

* `In Val` 은 해당 '점' 에서의 1차원 위치다. 위에선 `0.0` 은 시작점을 나타내고 `1.0` 은 끝 점 (생명주기가 다하는 점) 을 나타낸다.
* `Out Val` 은 이 모듈에서는 3차원 사이즈를 나타낸다.

위와 같은 선형으로 내려가는 커브는 `Curve Editor` 에서 확인할 수 있다. 다만 새로 만들어진 모듈 등의 커브를 확인하려면 `Emitters` 의 모듈 옆의 그래프 아이콘을 클릭해야 한다.

### 파티클 머터리얼의 컬러 다양성 주기

현재 `PS_Thruster` 파티클은 `M_Thruster` 머터리얼을 사용하고 있다. 그런데 파티클의 `Emitter` 의 `Module` 에서 색상을 줘서, 머터리얼에 적용하는 것 역시 가능하다.

![img26_particle_color]({{ "/assets/201807/img26_particle_color.JPG" | absolute_url }})

> [Emissive](https://docs.unrealengine.com/en-us/Resources/ContentExamples/MaterialNodes/1_5)

> [Use the Emissive Material Input](https://docs.unrealengine.com/en-us/Engine/Rendering/Materials/HowTo/EmissiveGlow)

그리고 `PS_Thruster` 파티클에서 `Initial Color` 을 사용해 컬러를 설정한다. 이 떄 컬러를 설정한다 하더라도 기존에 있던 `Color Over Life` 에서 컬러를 계속 갱신하기 때문에 이를 해제시켜서 기존의 컬러를 그대로 유지할 수 있도록 해야한다.

### 파티클 시스템 토글링하기

플레이어가 움직일 때에만 파티클을 뿜어낼 수 있도록 하고싶다. 그러면 `Blueprint` 에서 다음과 같이 사용할 수 있다.

![img26_particle_toggle]({{ "/assets/201807/img26_particle_toggle.JPG" | absolute_url }})

`Move` Function 에서 `Consume` 을 통해 이동 벡터를 뽑아내는 데, 이걸 활용해서 토글을 한다. 이동량이 $$ 0 $$ 이면 `Deactivate` 을, 그렇지 않으면 `Activate` 한다. (좀 더 안전하게 코딩한다면 플래그를 하나 추가해서 이미 `Activate` 되었는가를 확인했을 것이다.)

### `Burst` 파티클 만들어보기

`Cascade` 파티클 시스템은 `Burst` 을 지원한다. `Burst` 는 한꺼번에 여러 개의 파티클을 생성하는 기능이다. `Spawn` 모듈에서 이를 지원하고 있는데, 폭발 등의 효과에서 유용하게 사용할 수 있다.

![img26_particle_burst]({{ "/assets/201807/img26_particle_burst.JPG" | absolute_url }})

그리고, 파티클들의 루핑 횟수도 조절할 수 있다. 루핑 횟수를 조절하려면 `Required` 모듈에서 조절한다.

![img26_particle_duration]({{ "https://koenig-media.raywenderlich.com/uploads/2017/11/unreal-particles-23.JPG" | absolute_url }})

그러면 파티클 애니메이션이 다 끝난 다음에 `Viewport` 밑에 `[Completed]` 라고 뜰 것이다. 애니메이션이 완료됬다는 소리다.

이제 적이나 플레이어가 죽을 때 해당 파티클을 `Spawn` 시키면 된다. 

![img26_particle_spawn]({{ "/assets/201807/img26_particle_spawn.JPG" | absolute_url }})

#### `AActor` 의 정보에 따라 파티클 컬러를 바꿔보기

해당 프로젝트에서는 적은 여러가지 색상 타입을 가지고 있는데, 이 색상 타입에 따라서 파티클 `PS_Burst` 의 색상을 바꿔보고자 한다. `Cascade` 에서는 이를 지원한다.

* 그렇게 `Blueprint` 에서 해당 `Emitter` 의 여러가지 값을 변경하기 위해서는 각 모듈의 지정된 요소에 `Paramater Name` 이 특정 고유의 이름이어야 하며, 해당 요소는 `... Particle Parameter` 이어야 한다.

* 한 파티클의 각 `Emitter` 간의 정보를 공유하기 위해서는 정보를 공유할 `Emitter` 란을 오른쪽 클릭해서 `Emitter > Duplicate and Share Emitter` 을 사용하면 `+` 가 달린 각 `Module` 의 정보를 공유하는 이미터가 복제된다.

![img26_particle_share]({{ "/assets/201807/img26_particle_share.JPG" | absolute_url }})

하지만 여러 정보를 공유하면서도 특정 모듈에 대해서는 정보를 공유하고 싶지 않을 때는 해당 모듈을 `Delete` 하고 쓰면 다시 만들면 된다. 그러면 그 해당 `Module` 에 대해서는 `+` 가 붙지 않는다.

### TypeData Module?

사실 `Paritcle System` 을 만들 때 기본으로 주어지는 `Emitter` 는 `Sprite Emitter` 라고 하며, 다른 타입의 `Emitter` 을 만드는 것도 가능하다. 이 다른 타입의 `Emitter` 을 만들기 위해 사용하는 것이 `TypeData Module` 이다. 이 모듈들은 다른 타입의 파티클, 빔, 메쉬, 리본 등등을 방출하는 것이 가능하게끔 구현되어 있다고 한다.

> [https://docs.unrealengine.com/en-us/Engine/Rendering/ParticleSystems/Reference](https://docs.unrealengine.com/en-us/Engine/Rendering/ParticleSystems/Reference)

