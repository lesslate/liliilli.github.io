---
layout: post
title: "UE4 의 Behavior Tree 의 확률 분기형 Composite Node 만들기"
tags: 
date: 2018-07-28 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## 참고

> [[UE4] Behavior Tree の Composite Node を自作する](http://rarihoma.xvs.jp/2017/12/24/1/381#fn-381-1)

## `Composite` 노드란

`UE4` 의 `Behavior Tree` 에서 쓰이는 `Composite` 노드는, 여러 개의 노드 혹은 다수의 컴포지트를 가질 수 있는 각 브랜치의 루트로서 행동 트리의 행동들이 어떻게 실행될 것인지를 결정한다. `Task` 과 같이 `Decorator` 을 가질 수 있다.

문제는 그게 아니라... 기존 `UE4` 에서 제공하는 `Composite` 는 3 개 밖에 없다는 것이다. 물론 이걸로도 다 할 사람들은 하겠지만 꽤 제약이 있어보이는 건 사실이다.

1. `Sequence`
2. `Selector`
3. `Simple Parallel`

그런데 `UBTComposite` 클래스를 상속해서 `Composite` 에 필요한 구현 함수들을 잘 구현하면, `BP` 에서 직접 만든 `Composite` 을 사용해 쓸 수 있다고 한다. 여기서는 확률 분기에 따라 왼쪽 혹은 오른쪽의 행동을 결정하는 노드를 만들 예정이다.

## 작성 방법

우선 `UBTCompositeNode` 을 상속 (계승) 해서 `BTComposite_` 꼴로 시작하는 클래스 파일을 만든다. 조심해야 할 점은, Context Viewer 에서 C++ 파일을 생성할 때, 일반 템플릿을 사용하지 않고 검색을 통해 `BTComposite` 베이스 클래스를 지정해서 상속할 수 있도록 해야한다. 그렇지 않으면 제대로 구현한들 리플렉션이 반영되지 않아 다시 만들어야 한다.

> [UBTCompositeNode](http://api.unrealengine.com/INT/API/Runtime/AIModule/BehaviorTree/UBTCompositeNode/)

`BTComposite_WeightedRandom.h`

{% highlight cpp %}
UCLASS()
class SANDBOX_API UBTComposite_WeightedRandom : public UBTCompositeNode
{
  GENERATED_UCLASS_BODY()

  int32 GetNextChildHandler(
      struct FBehaviorTreeSearchData& SearchData,
      int32 PrevChild,
      EBTNodeResult::Type LastResult) const;

  FString GetStaticDescription() const override;

#if WITH_EDITOR
  bool CanAbortLowerPriority() const override;
  FName GetNodeIconName() const override;
#endif

public:
  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Weighted Random")
  float LeftChildSelectionRate;
};
{% endhighlight %}

`BTComposite_WeightedRandom.cc`

{% highlight cpp %}
UBTComposite_WeightedRandom::UBTComposite_WeightedRandom(
    const FObjectInitializer& ObjectInitializer) :
    Super(ObjectInitializer)
    , LeftChildSelectionRate(1.0f)
{
  NodeName = "Weighted Random";
  OnNextChild.BindUObject(
      this, 
      &UBTComposite_WeightedRandom::GetNextChildHandler);
}

int32 UBTComposite_WeightedRandom::GetNextChildHandler(
    FBehaviorTreeSearchData& SearchData,
    int32 PrevChild,
    EBTNodeResult::Type LastResult) const
{
  int32 NextChildIndex = BTSpecialChild::ReturnToParent;

  if (GetChildrenNum() == 2 && PrevChild == BTSpecialChild::NotInitialized)
  {
    const int32 LeftChildIndex = 0;
    const int32 RightChildIndex = 1;
    NextChildIndex = (FMath::Rand() <= LeftChildSelectionRate) ?
        LeftChildIndex :
        RightChildIndex;
  }

  return NextChildIndex;
}

FString UBTComposite_WeightedRandom::GetStaticDescription() const
{
  int32 ChildrenNum = GetChildrenNum();

  if (ChildrenNum == 2)
  {
    float LeftPercentage = LeftChildSelectionRate * 100;
    float RightPercentage = 100.0f - LeftPercentage;

    return FString::Printf(
        TEXT("Left : %.2f / Right : %2.f"),
        LeftPercentage, RightPercentage);
  }

  return FString::Printf(
      TEXT("Warning : Connect Just 2 Children Nodes (Currently %d Nodes"),
      ChildrenNum);
}

#if WITH_EDITOR

bool UBTComposite_WeightedRandom::CanAbortLowerPriority() const
{
  return false;
}

FName UBTComposite_WeightedRandom::GetNodeIconName() const
{
  return FName("BTEditor.Graph.BTNode.Composite.Selector.Icon");
}

#endif
{% endhighlight %}

그리고 `UE4 Editor` 쪽에서 컴파일을 눌러 리플렉션이 작동해서 `Composite` 에 반영이 될 수 있도록 해야한다.

![img28_weighted]({{ "/assets/201807/img28_weighted.JPG" | absolute_url }})

뙇.

## 분석 

### `GetNextChildHandler`

{% highlight cpp %}
int32 UBTComposite_WeightedRandom::GetNextChildHandler(
    FBehaviorTreeSearchData& SearchData,
    int32 PrevChild,
    EBTNodeResult::Type LastResult) const;
{% endhighlight %}

`GetNextChildHandler` 함수는 행동이 끝나고 나서 다음에 진행할 노드의 `Index` 를 반환하며, 해당 `Index` 을 내부의 `CurrentChild` 에 저장하는 함수이다. `UBTComposite` 클래스에서는 이 함수가 매우 중요하고 이 함수가 없으면 실행이 불가능하다.

`PrevChild` 는 이전에 실행된 노드의 `Index` 을, `LastResult` 는 마지막 노드에서의 실행 결과를 가지고 온다. 실행 결과에는 성공, 실패, 또는 다른 데코레이터 등에 의해서 중단 이 있을 수 있다.

여기서 중요한 점은, 노드의 `Index` 는 어디까지나 해당 노드에서 상대적인 인덱스를 나타내지, `BP` 에서 제공되는 실행 순서를 말하는 것이 아니다!

즉 다음과 같다.

![img28_specialchild]({{ "/assets/201807/img28_specialchild.JPG" | absolute_url }})

`BehaviorTreeTypes.h`

{% highlight cpp %}
namespace BTSpecialChild
{
  const int32 NotInitialized = -1;  
  // special value for child indices: needs to be initialized
  const int32 ReturnToParent = -2;  
  // special value for child indices: return to parent node
}
{% endhighlight %}

* `BTSpecialChild::ReturnToParent` (-2)
  는 현재 노드의 부모 노드를 가리키는 상대적 인덱스이다.
* `BTSpecialChild::NotInitialized` (-1)
  는 특정 노드를 가리키지는 않으나, 만약 `PrevChild` 가 이 값일 경우, 그리고 현재 이 노드로 이동해 왔을 경우에는 자식 노드에게 한번도 이동하지 않았음을 나타낸다. (정확하진 않음)
* 양수인 정수 인덱스
  자식의 상대적인 노드 인덱스를 가리킨다. 이 역시 왼쪽에서 오른쪽 순으로 0, 1, 2... 순으로 붙여진다.

따라서 위 Weighted Random Composite Node 의 경우에는...

{% highlight cpp %}
// 만약 Child 가 없으면 -2 을 반환하려 한다.
int32 NextChildIndex = BTSpecialChild::ReturnToParent;

// Child 는 항상 2 개여야 하고, 
// 아직 이번 페이즈에서는 임의 자식 노드에 한번도 들어가지 말았어야 한다.
if (GetChildrenNum() == 2 && PrevChild == BTSpecialChild::NotInitialized)
{
  const int32 LeftChildIndex = 0;
  const int32 RightChildIndex = 1;
  // UPROPERTY 로 지정한 확률 경계점에 대해 랜덤 확률로 왼쪽 혹은 오른쪽을 택한다.
  NextChildIndex = (FMath::Rand() <= LeftChildSelectionRate) ?
      LeftChildIndex :
      RightChildIndex;
}

// 리턴! 그러면 UE4 가 알아서 자식 노드 혹은 부모를 실행할 것이다.
return NextChildIndex;
{% endhighlight %}

### `CanAbortLowerPriority`

{% highlight cpp %}
bool UBTComposite_WeightedRandom::CanAbortLowerPriority() const
{
  return false;
}
{% endhighlight %}

이는 자식 노드 등지에서 `Decorator`를 가지고 있을 경우에, `Observer Aborts` 에서 `Lower Priority` 을 설정하는 것이 가능한지 아닌지를 플래그 반환하는 함수이다. 기본 `Composite` 에서는 `Sequence` 는 자식 행동들이 순서대로 진행됨을 보장해야 하기 때문에 `LowerPriority` 을 쓸 수 가 없다. 그렇게 하려면 함수에서는 `false` 을 반환한다. 하지만 `Selector` 의 경우, 자식 중 하나가 성공하면 끝인 노드이기 때문에 자식이 행동 중에 데코레이터에 의해 다른 하위 행동들을 중단시키게 할 수도 있다. 이 경우 `true` 을 반환한다.

그런데 지금 `Weighted Random` 의 경우, 둘 중 하나만을 선택해서 행동하기 때문에 `false` 을 반환해서 `Lower Priority` 을 비활성화 시킨다.

### `GetStaticDescription`

{% highlight cpp %}
FString UBTComposite_WeightedRandom::GetStaticDescription() const
{
  int32 ChildrenNum = GetChildrenNum();

  if (ChildrenNum == 2)
  {
    float LeftPercentage = LeftChildSelectionRate * 100;
    float RightPercentage = 100.0f - LeftPercentage;

    return FString::Printf(
        TEXT("Left : %.2f / Right : %2.f"),
        LeftPercentage, RightPercentage);
  }

  return FString::Printf(
      TEXT("Warning : Connect Just 2 Children Nodes (Currently %d Nodes"),
      ChildrenNum);
}
{% endhighlight %}

`BP` 에서 노드에 표시될 문구를 설정한다. 

## 결과

![img28_result]({{ "/assets/201807/img28_result.JPG" | absolute_url }})


