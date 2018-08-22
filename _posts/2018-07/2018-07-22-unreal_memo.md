---
layout: single
title: "UE4 메모 (4)"
tags: 
date: 2018-07-22 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

> [https://docs.unrealengine.com/en-us/Gameplay/Framework/GameMode](https://docs.unrealengine.com/en-us/Gameplay/Framework/GameMode)

## Game Mode

![img_gamemodebase]({{ "/assets/201807/img22_gmb.PNG" | absolute_url }})

* 진행되고 있는 게임들의 정보를 관리하는 메인 클래스가 두 종류가 있는데, 하나는 `Game Mode` 이고 또 하나는 `Game State` 이다.

가장 간단한 게임일지라도 기본적인 룰은 존재하며, 이 룰들이 `Game Mode` 을 만든다. `UE4` 는 본래 멀티 플레이어 FPS 게임을 전제로 해서 만들어진 엔진이기 때문에 `Game Mode` 는 기본 다음 규칙을 포함한다.

1. 한 게임에서의 플레이어와 관전자의 수, 최대 관람 수.
2. 한 게임에서 플레이어의 참전 수 제한, 팀 혹은 개개 인원마다 각기 다른 리스폰 지역 등을 관리
3. 게임이 **멈출 수 있는가, 없는가** 를 관리.
4. `UE4` 에서 지원하는 시네마틱 모드로 `Level` 을 진행할 지, 혹은 `Level` 간의 트랜지션 등을 관리

`UE4` 버전 4.14 이전에는 `AGameMode` 라는 종합적인 게임 모드 설정 관리자 클래스가 존재했지만, 이제는 `AGameModeBase` 라는 베이스가 되면서 게임의 설정 등을 심플하게 관리할 수 있는 클래스가 등장했다. 

`FPS` 및 멀티 플레이어를 구현할 때는 `AGameMode` 을 사용하는 것이 좋다고 한다.

* `AGameMode` `AGameModeBase` 역시 별개의 사용자 정의 클래스로 파생이 될 수 있다. 미션 타입, 스페셜 존이나 등등마다 하나씩 생성해서 각기 다른 게임 모드를 구현할 수 있다고 한다. 

* 하지만 각 레벨마다 하나의 `GameMode` 만을 사용할 수 있다. `AGameMode` 객체는 레벨이 불러와질 때 `UGameEngine::LoadMap()` 을 사용해서 만들어진다.

* 멀티 플레이어 게임에서 `AGameMode` 는 각각 클라이언트에 복제되지 않고 서버에서 하나만 가져서 동작해야 한다. 클라이언트는 `AGameMode` 의 실 객체의 정보를 볼 수 없고 제한된 `BP_` 블루프린트 객체 등으로만 접근이 가능하다. 만약 클라이언트가 현재 `AGameMode` 에 관련된 정보를 업데이트 해야한다면 `AGameMode` 와 같이 생성된 `AGameState` 에 정보를 저장해, 이를 각 클라이언트에 보낸다.

* `AGameModeBase` 와 `AGameStateBase`, 그리고 `AGameMode` 와 `AGameState` 는 따로 떨어질 수 없기 때문에 상속을 해서 커스텀 모드, 상태를 만든다고 하면 짝을 이뤄서 상속을 해야한다. 왜냐면 상태머신 등이 종속적으로 구현되어 있기 때문에...?

## Game State

규칙에 관련된 이벤트가 일어나고, 멀티 플레이어 등등의 게임에서 모든 플레이어 혹은 관전자가 해당 상태를 공유하기 위해서 사용되는 것이 `Game State` 이다. `AGameState` 계열은 다음 정보를 포함한다.

1. 게임이 진행된 시간
2. 각 플레이어의 상태들
3. 현재 게임 모드의 베이스 클래스 (?)
4. 게임이 시작됬는지 안됬는지를 확인

즉 `AGameState` 는 일종의 모니터 역할을 한다. 하지만 관리하는 정보는 개개인의 정보가 아니라 `AGameMode` 와 직접 관련된 정보들이어야 하고, 모두에게 보일 수 있어야 한다. 개개인의 정보를 관리할려고 하면 `Player State` 을 쓰면 된다고 한다..

## Input 키 바인딩 하는 방법

1. `Edit > Project Settings` 에 들어가서 `Input` 항목을 연다.
2. `Bindings` 에서 `Action Mappings` `Axis Mappings` 을 사용해 이름을 적고, 값을 설정해주면 된다.

* `Action Mapping` 의 경우, 누름 값과 뗌 값 사이의 중간 값이 없고 오로지 `Pressed` `Release` 상태만 존재한다. 총을 쏠 때와 같은 상황에서 사용될 수 있다.
* `Axis Mapping` 의 경우, `Axis value` 라는 것이 존재해서 (유니티의 그거 맞음) `Repeat` 상태가 존재하고 중간 값을 발췌할 수 있다. 조이스틱, 마우스와 같은 아날로그 신호를 캐치할 수 있다.

이름을 적고 나서 블루프린트에서 알아서 리플렉션을 통해 해당 키가 눌러졌는가 안 눌러졌는가, 값이 있는가에 대한 이벤트를 만들어준다.

어떤 `Pawn` 을 이동시키고자 할 때 적용할 수 있는 `BP` 도안은 다음과 같다.

![img_bp]({{ "/assets/201807/img22_bp.PNG" | absolute_url }})

> `Tick()` 에서 델타 타임을 가지고 와서 적용하는 것이 중요하다.

## Basic Actor Collisions

> [https://docs.unrealengine.com/en-US/Engine/Physics/Collision/Overview](https://docs.unrealengine.com/en-US/Engine/Physics/Collision/Overview)

> [https://docs.unrealengine.com/en-us/Engine/Physics/Collision/Reference](https://docs.unrealengine.com/en-us/Engine/Physics/Collision/Reference)

`AActor` 들이 서로 충돌하기 위해서는 충돌할 수 있는 면적 혹은 공간이 따로 설정되어야 한다. 이 경우 다음과 같이 나뉜다.

* `Collision Mesh` 는 메쉬가 `UE4` 로 임포트될 때 자동으로 생성해준다. 혹은 `UE4` 에서 지원하는 것을 쓰지 않고 따로 외부 소프트웨어에서 충돌 메쉬를 만들어서 임포트할 수도 있다. 

* `Collision Component` 는 `box` `capsule` `sphere` 등과 같은 간단한 메쉬를 사용해서 충돌체를 만든다.

그리고 충돌을 제대로 계산하기 위해서 `StaticMesh` 을 루트로 옮겨야 한다.

* `UE4` 는 기본으로 `Root Component` 의 충돌체만을 충돌 시뮬레이션에 입력한다. 사실 `Root Component` 아래에서 메쉬를 만들어서 충돌체를 구현하면, 움직이지 않고 다른 오브젝트가 충돌할 경우에는 해당 충돌체 역시 계산범위에 들어가나, 자기 자신이 이동할 경우에는 제대로 계산이 되지 않는다.

![img_sweep]({{ "/assets/201807/img22_sweep.PNG" | absolute_url }})

그리고 `Unity` 와는 다르게 각 메서드마다 물리엔진을 처리할지 말지 결정하는 것이 있는데, `Sweep` 도 그 중 하나다. 이걸 체크하면 `Root component` 의 충돌체에 대해 원래 점에서 이동한 점까지의 레이 트레이싱 충돌 시뮬레이션을 한다.

### Collision Response 결정하는 법

> [https://docs.unrealengine.com/en-us/Engine/Physics/Collision/Reference](https://docs.unrealengine.com/en-us/Engine/Physics/Collision/Reference)

![img_collision]({{ "/assets/201807/img22_colllision.png" | absolute_url }})

* 각 Mesh 의 충돌체는 `Object Type` 을 가진다. 이 오브젝트 타입은 커스터마이징 할 수 있다. 아무튼 `Object Response` 에서 다른 오브젝트에 대해 자기 충돌체가 어떤 타입으로 반응을 할 지를 결정할 수 있다. 위에서는 다른 `WorldDynamic` 충돌체에 대해 `Overlap` 으로 처리하고자 했다.

* 혹은 프리셋을 사용해서 간편하게 모든 레이어에 대한 설정을 바꿀 수 있다.

* `Overlap` 이 되었을 때 event 는 `OnComponentBeginOverlap` 이며 `Block` 상태에서는 `Hit` 이벤트가 호출될 것이다.

## Creating Material Parameters

![img_material]({{ "/assets/201807/img22_material.png" | absolute_url }})

위의 단축키를 알고 가자.

* 각 `Material` 에 대해 `Parameter` 을 만들어서 `Uniform` 같이 넘겨줄 수 있다. 또한 `Constant` 을 `Parameter` 로 만들 수도 있다. 그런데 문제점은, 여기서 생성된 `Parameter` 는 런타임에서 동적으로 값을 바꿀 수가 없고 오로지 정적으로, 파생된 머터리얼들을 생성할 때에만 값을 변경할 수 있다.

* `Level` 에 생성된 각 `Actor` 객체에 대해서 `Material` 을 바꿀 수도 있다.

런타임에 머터리얼의 특성을 바꾸기 위해서는 `Dynamic Material Instance` 을 사용하는 수 밖에 없다.

### Material Instance Dynamic

> [http://api.unrealengine.com/KOR/Engine/Rendering/Materials/MaterialInstances/](http://api.unrealengine.com/KOR/Engine/Rendering/Materials/MaterialInstances/)

이전의 머터리얼은 컴파일 시간에 계산되기 때문에 `Material Instance Constant` 라고 하며 지금 말하는 것은 런타임에 계산이 되기 때문에 `Dynamic` 이라 말한다. `MID` `MIC` 라고 부르기도 한다.

![img_material]({{ "https://koenig-media.raywenderlich.com/uploads/2017/06/29-1.gif" | absolute_url }})

1. `MID` 을 생성하려면 우선 파라미터화된 머터리얼 혹은 머터리얼 인스턴스 콘스턴트를 사용한다.
2. 블루프린트에서 패러미터가 있는 머터리얼을 받아서 `Create Dynamic Material Instance` 노드를 사용해서 물려준다. 
3. 꼬리를 물어서 패러미터 값을 변경시킨다.
4. `Set Material` 을 사용해서 오브젝트에 만들어진 `MID` 을 적용시킨다.

다음은 `M_Cube` `MIC` 을 사용해서 `Cube Material` 이라는 `MID` 를 생성해, 거기의 `ColorAlpha` 라는 값을 변경하게끔 하는 블루프린트 노드 그래프다.

![img_blueprint]({{ "/assets/201807/img22_mid.png" | absolute_url }})

