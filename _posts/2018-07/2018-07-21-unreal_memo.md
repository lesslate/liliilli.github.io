---
layout: single
title: "UE4 Viewport 단축키들 간단 메모"
tags: 
date: 2018-07-21 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## Viewport

![img_viewport]({{ "/assets/201807/img20_view.PNG" | absolute_url }})

* 마우스 `오른쪽` 클릭 후, 커서 이동 : 해당 카메라 시점에서 이동하지 않고 시야 돌려보기

* 마우스 `오른쪽` 클릭 후, `ALT` 눌러 이동 : 스무스하게 줌 인 / 줌 아웃.

* 마우스 `오른쪽` 클릭 후, `WASD` 눌러 이동 : Viewport 의 카메라를 이동시킨다.

* 마우스 `왼쪽` 클릭 후, `ALT` 눌러 이동 : 해당 지점 혹은 액터를 고정으로 회전.

* `F` 키 : 해당 액터에 대해 포커스를 맞춰서 줌 인.

* `F11` 키 : Viewport 을 풀스크린으로 키운다. (다시 누르면 창으로 돌아감)

> 그 외 라이팅을 끄거나 와이어프레임만을 나타나게 하는 기능들이 존재하나, 자세한 것은 [다음 주소](https://docs.unrealengine.com/en-US/Engine/UI/LevelEditor/Viewports/ViewModes) 를 볼 것.

## 액터를 클릭하고 나서...

* `END` 키 : 무언가가 닿을 때까지 하단으로 이동시킴.

* `CTRL + W` 키 : 액터 등을 복제함.

## `Brush` 로 특정 메쉬를 만들고자 할 때

![img_subtractive]({{ "/assets/201807/img21_subtract.PNG" | absolute_url }})

`Brush` 는 `StaticMeshActor` 에게 영향을 미치지 않는다. `Brush` 는 `Brush` 끼리만 작용한다. 그리고 특정 모양을 만들 때 해당 `Brush` 의 `Order` 역시 중요하다. `Subtractive` 로 모양을 뺄 경우에는 해당 `Brush` 의 `Order` 을 `To Last` 로 하면 좋다.

그리고 `Brush` 는 임시적으로 만들 떄 쓰이며 최종적으로는 `StaticMesh` 로 변환해서 써야 한다. 이 경우 `Details` 의 `Brush settings` 에서 정적 메쉬를 만들 수 있다.

그렇게 하면 다음과 같이 중간이 텅 빈 정적 메쉬가 만들어 진다.

![img_sm]({{ "/assets/201807/img21_sm.PNG" | absolute_url }})