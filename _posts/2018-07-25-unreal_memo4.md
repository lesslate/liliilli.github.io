---
layout: post
title: "UE4 메모 (7) 사운드 등"
tags: 
date: 2018-07-25 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## Simple Game Tutorial in UE4

> [https://www.raywenderlich.com/168091/create-simple-game-unreal-engine-4](https://www.raywenderlich.com/168091/create-simple-game-unreal-engine-4)

### Audio and Sound

> [https://docs.unrealengine.com/en-US/Engine/Audio](https://docs.unrealengine.com/en-US/Engine/Audio)

`Content Viewer` 에서 사용하는 음원들은 기본적으로 `Looping` 이 빠져있다. 따라서 블루프린트를 만졌던 것 처럼 음원을 클릭해서 `Looping` 을 설정해줘야 루프가 가능하다.

### `Animation Notification`

> [https://docs.unrealengine.com/en-us/Engine/Animation/Sequences/Notifies](https://docs.unrealengine.com/en-us/Engine/Animation/Sequences/Notifies)

`AnimNotifies` 을 사용해서 애니메이션 시퀀스 도중에 특정 시점에서 이벤트 등이 발생할 수 있도록 할 수 있다. 흔한 예시로는 발걸음 애니메이션 도중에 발자국 소리가 들리게 한다던가, 칼을 휘두르는 애니메이션에서는 휘두르는 시점에 바람 소리가 나게 하는 것이 있다. 또는 파티클 시스템도 이벤트에 접목이 가능하다.

![img_example1]({{ "https://koenig-media.raywenderlich.com/uploads/2017/10/05.gif" | absolute_url }})

혹은 이런 내부 기능 말고도 직접 `AActor` 와 관련된 것들 (`AWidget` 등..) 을 Spawn 하게 할 수도 있다! 

우선 `SK_Muffin_Walk` 의 `Animation Blueprint Detail Viewer` 을 열면, 하단에 `Notifies` 라고 하는 부분이 있다.

![img_notifytrack]({{ "https://docs.unrealengine.com/portals/0/images/Engine/Animation/Sequences/Notifies/PlayParticleEffectNotify.png" | absolute_url }})

그리고 `Notifies` 에서 밝은 부분을 `Notify Track` 이라 말한다. 프레임을 옮길 때는 하단의 숫자가 적혀진 부분의 붉은 커서를 옮기면 된다. 

그 후에 `Notify Track` 에서 오른쪽 클릭을 한 뒤에 `Particle` 혹은 `Sound` 그리고 또 다른 `State` 등을 생성해서 적용할 수 있나보다.

`Notify Track` 에 추가된 요소는 옆의 `Detail View` 에서 설정을 변경할 수 있다.

### `Sound Cue`

> [https://docs.unrealengine.com/en-us/Engine/Audio/SoundCues/Editor](https://docs.unrealengine.com/en-us/Engine/Audio/SoundCues/Editor)

발자국 소리를 적용하고, 게임 뷰에서 테스트를 해보면 발이 땅에 닿을 떄마다 `Notify Track` 의 플래그가 작동해서 소리가 나는 것을 확인할 수 있다. 하지만 하나같이 다 같은 발자국 소리라서 진부하다.

* 이럴 때 `Sound Cue` 을 사용해서 소리를 합치거나 음향 효과를 줘서 다양한 효과를 낼 수 있게 할 수 있다. 또한 `Sound Cue` 자체를 `UE4` 에서 사용 가능한 자체 사운드 음원으로도 취급할 수 있다.

* `Sound Wave` 는 `Sound Cue Editor` 에서 임포트된 오디오 파일을 말한다.

* `Sound Cue` 을 만들기 위해서는 `Content Viewer` 의 임의 일반 오디오 파일을 오른쪽 클릭해서 `Create Cue` 을 눌러 postfix 가 `_Cue` 인 `Cue` 파일을 만들 수 있다.

### 사운드 피치 및 볼륨 자동 조절하기

`Sound Cue` 를 더블클릭해서 블루프린트를 열면, `Graph` 로 여러가지 효과를 낼 수 있도록 정할 수 있다. 다만 이 블루프린트 그래프에서는 사운드에 관련된 것만 넣을 수 있고, 그 외의 블루프린트 노드는 삽입할 수 없다.

따라서 다음과 같이 `Modulator` 노드를 삽입해서 피치 등을 조절할 수 있게 되었다.

![img_modulator]({{ "/assets/201807/img25_cue1.JPG" | absolute_url }})

이렇게 만들어진 `Cue` 는 일반 사운드와 더불어 `Animation BP`, 일반 `Blueprint` 등에서 사용할 수 있다.

![img_soundbp]({{ "/assets/201807/img25_soundbp.png" | absolute_url }})

여기서 주로 쓰이는 것은 다음 두 가지이다.

* `Play Sound 2D` 는 감쇠, 음향학에서 적용되는 여러가지 효과없이 사운드를 출력한다.

* `Play Sound at Location` 은 감쇠, 음향학 등에서 적용되는 여러가지 효과를 동반해서 현재 플레이어 `Pawn` 과 비교하여 소리를 출력한다.

### `Spatialization`

> [http://api.unrealengine.com/INT/API/Runtime/Engine/IAudioSpatialization/index.html](http://api.unrealengine.com/INT/API/Runtime/Engine/IAudioSpatialization/index.html)

`Spatialization` 은 3D 공간에서 사운드 음향 위치에 따라 스피커에 각기 다른 비중으로 음향을 내보내는 기법이다. 이 `Spatialization` 이라는 기법은 FPS 등에서 많이 사용한다.

`Sound Cue` 에서 `Spatialization` 을 활성화는 방법은 두 가지가 존재한다.

* `Sound Attenuation asset` 은 `Attenuation` 및 `Spatialization` 이 미리 설정된 것으로, 직접 설정을 만질 필요없이 체크만으로 간단하게 설정이 가능하다. 기본적으로 설정이 되어있는 상태이다.

* `Override Attenuation` 은 직접 설정을 해서 감쇠, 공간 음향을 설정한다. 기본 설정을 해제하면 된다.

그리고 나서 다시 테스트를 하면 이제는 `Spatializaion` 이 적용되어 공간에 따라 음향이 스피커에 다르게 들린다.

* 주의할 점은, 기본으로 `Camera` 가 `Audio Listener` 이기 때문에 카메라 시점에서 3D 음향의 공간 효과가 적용된다. 따라서 카메라 말고도 다른 컨트롤러를 가지는 `Pawn` 등에 공간 효과를 적용한다고 하면, `BP` 에서는 `Set Audio Listener Override` 을 사용해야 하면 된다.

> [https://answers.unrealengine.com/questions/28880/change-audio-listener-position.html](https://answers.unrealengine.com/questions/28880/change-audio-listener-position.html)

> [UE4初心者が頑張ってるブログ](http://mozpaca.hatenablog.com/entry/20180225/1519521954)

### `Attenuation` 및 `Timeline` 을 사용한 감쇠

* `AActor` 블루프린트 안에서 `Audio Component` 을 만드는 것도 가능하다. 다만 이 경우에는 `Audio` 컴포넌트는 자동으로 `Activated` 가 되어있기 때문에 `Detail Viewer` 에서 끄는 것이 필요하다.

그리고 `Audio Component` 는 디테일 뷰어에서 감쇠의 거리를 설정하는 것도 가능하다.

![img_soundbp]({{ "/assets/201807/img25_atten1.JPG" | absolute_url }})

1차 거리, 2차 경계점 거리 안에서는 볼륨이 페이드가 될 것이다. 하지만 지금 이 프로젝트에서는 구름이 화면 밖의 밑쪽 컬리젼에 들어갈 때 자동으로 `AActor` 의 인스턴스를 사라지게 하는데, 이 때 `APawn` 이 감쇠 거리에 있다면 컷아웃이 되버린다.

이 컷아웃을 방지하기 위해서 `AActor` 가 제공하는 이벤트인 `FadeOut` 에서 이 경우를 대비해 볼륨의 페이드 아웃을 설정해야 한다.

![img_fadeout]({{ "/assets/201807/img25_fadeout1.JPG" | absolute_url }})

`Set Volume Multiplier` 와 `Timeline` 을 사용해 볼륨을 프레임마다 조절하면서 객체를 사라지게 할 수 있다.

> [Timeline](https://docs.unrealengine.com/en-us/Engine/Blueprints/UserGuide/Timelines)

### Sound Class 와 Sound Mix

![img_soundclass]({{ "https://koenig-media.raywenderlich.com/uploads/2017/10/28.JPG" | absolute_url }})

* `Sound Class` 는 여러 개의 사운드를 그룹화하기 위한 간단한 방법이다. 그리고 `Sound Class` 는 각 그룹별의 볼륨, 피치를 조절하는 것이 가능한데 이 때 `Sound Mix` 라는 것이 필요하다. 

* `Sound Mix` 는 기본적으로 테이블로 구성되며 각 테이블의 레이블마다 하나의 `Sound Class` 의 정보로 구성된다.

이걸 만들어본다.

그리고 사운드 블루프린트에 대해 값이 변경됬을 때의 `Override` 할 수 있는 함수를 오버라이딩해서 `Set Sound Mix Class Override` 을 사용해서 해당 사운드 믹스의 볼륨을 조정하게 한다.

그 후에, `Event Pre Construct` 와 `Push Sound Mix Modifier` 을 사용해서 볼륨을 적용을 한다.