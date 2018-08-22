---
layout: single
title: "UE4 메모 (5) UMG"
tags: 
date: 2018-07-22 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

> [http://api.unrealengine.com/KOR/Engine/UMG/](http://api.unrealengine.com/KOR/Engine/UMG/)

> [https://docs.unrealengine.com/en-us/Engine/UMG](https://docs.unrealengine.com/en-us/Engine/UMG)

## UMG UI (Unreal Motion Graphics UI Designer)

`UMG` 는 `Unity` 의 캔버스에서 `UI` 을 구성하는 것과도 같으며, 게임 안의 `HUD` 등을 구현하는데 사용될 수 있다. `UMG` 의 코어는 위젯들로 구성되어 있으며 이는 인터페이스를 구축하는데 사용될 수 있는 미리 만들어진 함수들로 구성된 타입이다.

`UMG` 의 위젯들은 특별하게 만들어진 블루프린트 에디터로 편집을 한다.

1. `Designer` 는 인터페이스 및 레이아웃을 만든다.
2. `Graph` 는 사용하고 있는 위젯 뒷편의 메커니즘을 구현한다.

### 각종 사항들

#### `UMG` 블루프린트 생성하기

![img_umg]({{ "/assets/201807/img22_umg.png" | absolute_url }})

#### Anchor

> [https://docs.unrealengine.com/en-US/Engine/UMG/UserGuide/Anchors](https://docs.unrealengine.com/en-US/Engine/UMG/UserGuide/Anchors)

`Anchor` 는 UI 위젯을 캔버스 패널위에 적절한 위치에 고정시킬 뿐만 아니라, 해상도가 변해도 기본 위치를 고정시킬 수 있도록 한다. `Anchor` 의 값은 $$ \text{Min}(0, 0) $$ 에서 $$ \text{Max}(1, 1) $$ 까지 위치하는데, 이 때 좌상단이 $$ (0, 0) $$ 이 된다.

![img_medalion]({{ "https://docs.unrealengine.com/portals/0/images/Engine/UMG/UserGuide/Anchors/AnchorMedallion.png" | absolute_url }})

노란색 박스에 있는 것이 `Anchor Medalion` 이라고 한다. 이를 조절해서 8 방향, 혹은 스트레치된 상태 및 아닌 상태로 `UI` 의 위젯을 고정시킬 수 있다.

### `UMG` 에서의 Bindings 

![img_bind]({{ "/assets/201807/img22_bind.png" | absolute_url }})

해당 위젯의 `Appearance` 에 있는 `Bind` 마크다운을 눌러서 `Create Binding` 을 하면 매 프레임마다 해당 값을 어딘가에서 받아서 갱신할 수 있도록 이벤트를 만든다.

만약에 `BP_GameManager` 에서 타이머를 가져와서 이를 매 프레임마다 표시한다고 하면,

![img_bindfunction]({{ "/assets/201807/img22_bindfunction.png" | absolute_url }})

과 같이 적용하면 된다. 물론 `GameManager` 는 다른 곳에서 레퍼런스를 지정을 해줘야 할 것이다.

