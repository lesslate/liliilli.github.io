---
layout: post
title: "UE4 C++ 파일 Delete 시에 필요한 과정들"
tags: 
date: 2018-07-28 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## UE4 에서 C++ 파일을 지우기 위해서는...

> [Cannot open source file name.generated.h](https://answers.unrealengine.com/questions/740950/cannot-open-source-file-namegeneratedh.html)

`UE4` 는 C++ 파일에 여러가지 복잡한 리플렉션 등을 적용하고 있으며, 또한 C++ 파일 자체에 종속된 여러가지 자동 생성된 헤더 파일 등이 있어서 무턱대고 C++ 에디터에서 함부로 파일을 지웠다가는 컴파일 에러가 터져버린다.

다음과 같이 순서를 거치면 거의 왠만해서는 해결이 된다고 한다.

1. UE4 에디터와 비주얼 스튜디오를 닫는다.
2. 문제를 일으키는 `.cpp` 파일과 `.h` 파일을 디스크에서 제거한다.
3. 프로젝트 폴더에서 `.vs` `Binaries` `Intermediate` `Saved` 파일과 `.sin` 솔루션 파일을 제거한다.
4. 프로젝트의 `.uproject` 파일을 오른쪽 클릭해서 Generate Visual Studio Project Files 을 클릭해 다시 솔루션 파일을 만든다.
5. `.sin` 에서 수동으로 재빌드 하거나 아니면 에디터를 열어서 버전에 맞게 다시 빌드를 시킨다.

## C++ 에서 소스 작성 후 컴파일 시, `LNK2001` 에러 처리.

> [Link errors in IGameplayTaskOwnerInterface](https://answers.unrealengine.com/questions/429455/link-errors-in-ubttasknode-classes.html)

{% highlight cpp %}
2>AllowNextMarker.cpp.obj : error LNK2001: unresolved external symbol "public: virtual void __cdecl IGameplayTaskOwnerInterface::OnGameplayTaskActivated(class UGameplayTask &)" (?OnGameplayTaskActivated@IGameplayTaskOwnerInterface@@UEAAXAEAVUGameplayTask@@@Z)
{% endhighlight %}

컴파일 시 다음과 같이 해당 클래스에서 구현해야 할 함수가 아닌데도 불구하고 저렇게 `IGameplayTaskOwnerInterface` 에 대해 `LNK2001` 에러가 나오면, 해당 프로젝트의 설정 파일인 `.Build.cs` 파일에 `GameplayTasks` 을 추가하면 된다.

{% highlight csharp %}
public Sandbox(ReadOnlyTargetRules Target) : base(Target)
{
	PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
	PublicDependencyModuleNames.AddRange(new string[] {
			"Core",
			"CoreUObject",
			"Engine",
			"InputCore",
			"GameplayTasks" });
	PrivateDependencyModuleNames.AddRange(new string[] {  });

    // ...
}
{% endhighlight %}

그러면 `LNK2001` 에러가 사라지고 정상적으로 컴파일이 된다.

## `GENERATED_BODY()` vs `GENERATED_UCLASS_BODY()` ?

`UCLASS` 프로퍼티가 붙은 `UE4` 에 관련된 모든 타입들은 리플렉션 등을 위해서 `GENERATED_BODY()` 과 같은 매크로를 붙인다.

그런데 `_UCLASS_` 가 있는 매크로를 사용하게 되면 해당 클래스는 규격상 정해진 `Constructor` 을 직접 만들어 함수 바디를 독립적으로 구현해야 한다. 

{% highlight cpp %}
UBTComposite_WeightedRandom::UBTComposite_WeightedRandom(
		const FObjectInitializer& ObjectInitializer) :
		Super(ObjectInitializer)
		, LeftChildSelectionRate(1.0f)
{
	NodeName = "Weighted Random";
	OnNextChild.BindUObject(this, &UBTComposite_WeightedRandom::GetNextChildHandler);
}
{% endhighlight %}

