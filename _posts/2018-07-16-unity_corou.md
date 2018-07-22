---
layout: post
title: "Unity 5 코루틴 메모."
tags: 
date: 2018-07-16 +0900
categories: Unity
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

![img_coroutine]({{ "/assets/201807/img16_coroutine.jpg" | absolute_url }})

`Unity` 에서 코루틴은 주로 타이머 콜백 등에 많이 쓰인다. 다만 `StartCoroutine()` 및 `StopCoroutine` 을 사용할 때 여러가지 방법으로 콜백을 넘겨줄 수 있다.

* `IEnumerator` 을 반환으로 하는 콜백 함수의 이름을 `string` 으로 넘겨서 리플렉션을 통해 코루틴을 실행하는 방법
* `IEnumerator` 을 반환으로 하는 콜백 함수를 코루틴 바인딩, 정지시에 곧바로 실행시켜서, 반환된 `IEnumerator` 을 사용하는 방법
* `IEnumerator` 을 반환으로 하는 콜백 함수 자체를 미리 호출해서 `IEnumerator` 을 해당 `MonoBehaviour` 에 객체 변수로 저장하고, 필요할 때마다 콜해서 실행하는 방법.

지금은 2 번째와 3 번째의 방법이 완전히 선호된다.

> [yield return에서 사용할 수 있는 것들](http://theeye.pe.kr/archives/2725)