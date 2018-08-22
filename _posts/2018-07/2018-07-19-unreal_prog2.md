---
layout: single
title: "UE4 ITCP 2 메모"
tags: 
date: 2018-07-17 +0900
categories: UE4
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

> [https://docs.unrealengine.com/en-us/Programming/Introduction](https://docs.unrealengine.com/en-us/Programming/Introduction)

## Object/Actor Iteratos

* `UObjectIterator.h` 에서 제공되는 이터레이터인 `TObjectIterator<>` 을 사용해서 특정 `UObject` 타입들에 해당되는 객체들을 쉽게 순회할 수 있다.

{% highlight c++ %}
#include "UObjectIterator.h"

// ...

for (TObjectIterator<UObject> It; It; ++It) {
  UObject* CurrentObject = *It;
  UE_LOG(LogTemp, Log, TEXT("Found UObject named :%s"), 
      *CurrentObject->GetName());
}
{% endhighlight %}

`UObject` 대신에 `UObject` 을 상속하는 다른 타입들 역시 들어갈 수 있다. `AActor` 류 역시 상속된 자식 클래스이기 때문에 사용할 수는 있지만 문제점이 있다.

> `PIE` (Play in Editor) (툴 내에서 게임을 플레이하는 뷰) 안에서 `TObjectIterator<>` 을 사용하는 것은 예기치 못한 결과를 낳을 수 있다. 왜냐면 게임 뿐만 아니라, 에디터 내부에서 사용되는 `UObject` 타입 역시 가져올 수 있기 때문이다.

* `AActor` 계만을 이터레이팅 하는 `TActorIterator` 는 위 `TObjectIterator` 의 문제점을 갖고 있지 않다. `TActorIterator` 는 다른 헤더 파일에서 구현하고 있는데 경로가 복잡하기 때문에 인텔리센스의 힘을 빌리자.
  * 또한 `TActorIterator` 는 생성시에 `UWorld` 인스턴스의 포인터를 받는다.

> When creating an actor iterator, you need to give it a pointer to a UWorld instance. Many UObject classes, such as APlayerController, provide a GetWorld method to help you. If you are not sure, you can check the ImplementsGetWorld method on a UObject to see if it implements the GetWorld method.

{% highlight c++ %}
APlayerController* MyPC = GetMyPlayerControllerFromSomewhere();
UWorld* World = MyPC->GetWorld();

// Like object iterators, 
// you can provide a specific class to get only objects that are
// or derive from that class
for (TActorIterator<AEnemy> It(World); It; ++It) {
    // ...
}
{% endhighlight %}

## Basic Memory Management & GC

`UE4` 는 GC 을 구현하기 위해 리플렉션 시스템을 사용하고 있다. 이를 사용해서 메모리 관리를 쉽게 할 수 있다. 다만 언리얼의 GC 의 혜택을 받기 위해서는 일련의 고정된 프레임을 만드는 것이 필요하다고 한다.

대개 이렇게 코드를 짜서 `GC` 바인딩을 하도록 한다.

{% highlight c++ %}
UCLASS()
class MyGCType : public UObject {
  GENERATED_BODY()
};
{% endhighlight %}

`UObject` 상속으로 제대로 만들어진 타입은 질의 시에 `Root Set` 안의 오브젝트에서 다른 오브젝트로 프로퍼티를 통해 참조(`Reference`)할 수 있는 경로가 있으면 `GC` 되지 않는다. `UE4` 는 GC 컬렉터를 일정 주기로 돌리고 있다고 한다.

만약 다음과 같은 코드가 있다고 하면

{% highlight c++ %}
void CreateDoomedObject() {
  MyGCType* DoomedObject = NewObject<MyGCType>();
}
{% endhighlight %}

`MyGcType` 이 `UObject` 을 상속한다고 했을 때, 힙 오브젝트 포인터를 생성해서 그것의 객체 포인터를 변수로 받는다. 하지만 `UPROPERTY` 에 의해 구현된 임의 객체의 멤버 변수로 저장하지는 않기 때문에 (즉 `Root set` 에 없음) `GC` 가 구동되면 해당 객체는 힙 소멸이 될 것이다.

* `NewObject<TType extends UObject>` 는 `GC` 가능한 `UObject` 팩토리 메서드이다. 자세한 것은 이 [문서](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Objects/Creation) 를 참조한다.

## Actors and GC

* `AActor` 로 파생되는 타입들은 자동으로 `GC` 되지 않기 때문에 한번 `AActor` 가 Spawn 되면 해당 객체를 `.Destroy()` 해줘야 한다.

* 또한 다른 케이스로, `Root set` 에 바인딩해서 제멋대로 `GC` 되지 않기 위해서는 포인터 변수 등에 `UPROPERTY()` 을 붙여줘야 한다. 또한 어느 한 포인터에 바인딩 된 `UObject` 가 `Destroy()` 될 때, `UPROPERTY()` 레퍼런스들은 safe 하게 `nullptr` 값으로 초기화된다.

{% highlight c++ %}
if (MyActor->SafeObject != nullptr) {
  // Use SafeObject
}
{% endhighlight %}

* `AActor` 의 `GC` 에 대해서 주의할 점은, `Destroy()` 을 사용한다고 하더라도 다음 페이즈의 `GC` 가 컬렉팅하기 전 까지는 계속 살아있게 된다. 그래서 `IsPendingKill()` 이라는 것을 사용하면 해당 `UObject` 가 삭제를 기다리고 있는지를 알 수 있다. 이 함수를 잘 활용하자.

## UStruct and GC

* `UStruct` 는 `UObject` 의 경량 버전으로 의도되서 설계되었는데, 그래서 `UStruct` 은 `GC` 가 되지 않는다. 따라서 동적으로 만들려고 하면 스마트 포인터를 사용하자.

## 非UObject 의 UE4 Reference

* `UObject` 을 상속하지 않는 비UObject 객체 타입을 `UE4` 의 `GC` 와 `Root set`에 바인딩시킬려고 한다면, `FGCObject` 을 상속하고 그 안의 `AddReferencedObjects` 을 구현하면 된다.

{% highlight c++ %}
class FMyNormalClass : public FGCObject {
public:
  UObject* SafeObject; 
 
  FMyNormalClass(UObject* Object)
      : SafeObject(Object) {
  } 
  void AddReferencedObjects(FReferenceCollector& Collector) override {
    Collector.AddReferencedObject(SafeObject);
  }
};
{% endhighlight %}

## 클래스 네이밍 Prefix

| Base Type                       | Prefix |
| ------------------------------- | ------ |
| Actor                           | A      |
| Object                          | U      |
| Enums                           | E      |
| Interface                       | I      |
| Template                        | T      |
| Class derived Slate UI(SWidget) | S      |
| Otherwise                       | F      |

## UE4 에서 써야할 정수형 타입

`short` `int` `long` 등등은 플랫폼마다 구현 사항이 다 다르기 때문에 `int8` `int16` `int32` `int64` 와 같은 타입을 쓴다.

이 때 Unsigned 는 쓰지 않는 것이 좋다. 그리고 `UE4` 에서 구현된 타입에 대해 한계치를 알려면 [TNumericLimits<>](http://api.unrealengine.com/INT/API/Runtime/Core/Math/TNumericLimits/index.html) 을 본다.

## Strings

`UE4` 에서는 4 개의 스트링을 구현하고 있다.

1. [`FString`](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/StringHandling/FString) 은 `TEXT()` 매크로로 만들 수 있으며 도중 변경이 가능한 스트링 클래스다. 길이를 늘리다던가 서브스트링을 만든다던가 역으로 돌린다던가 하는 것이 가능하다. 또한 `FString` 은 다른 스트링 타입과 비교될 수 있는데, 문제는 변환 비용이 생긴다는 것이 흠이다.
  * 일반 상황에서는 `FString` 을 매크로를 사용해서 쓰자.

2. [`FName`](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/StringHandling/FName) 은 변경 불가능하지만 재사용성이 보장되는 (다른 곳에서 같은 텍스트를 사용해도 하나의 텍스트만을 지칭하게 됨) 스트링 타입이다.
  * 따라서 컨텐츠 브라우저, 머터리얼 인스턴스의 속성을 바꾼다거나, 컴포넌트의 이름을 지정할 때에 유용하게 사용된다.
  * 이 타입은 `Case-insensitive` 하기 때문에 대소문자가 구별이 안된다. 

3. [`FText`](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/StringHandling/FText) 은 `NSLOCTEXT` 매크로를 사용하며 게임 상에 보여주기 위해서 사용되는 스트링 타입이다. `FText` 는 런타임 로컬라이제이션, 스트링의 룩업 테이블 저장이 된다.
  * 물론 로컬라이제이션을 고려하지 않은 텍스트도 `NSLOCTEXT` 을 사용해 저장할 수 있긴 하다. 
  * 미리 `LOCTEXT_NAMESPACE` 을 사용해서 네임스페이스를 적용하면 `LOCTEXT` 을 사용해도 된다.

{% highlight c++ %}
FText TestHUDText = NSLOCTEXT("Your Namespace", "Your Key", "Your Text");
{% endhighlight %}

{% highlight c++ %} 
#define LOCTEXT_NAMESPACE "Your Namespace" 
// ... 

FText TestHUDText = LOCTEXT( "Your Key", "Your Text" ) 

// ... 
#undef LOCTEXT_NAMESPACE 
{% endhighlight %}

이렇게 쓰면 `UE4` 의 로컬라이제이션 시스템에 적용이 된다.

4. [`TChar`](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/StringHandling/CharacterEncoding) 은 `UE4` 의 내부에서 사용하고 있는 `UTF-16` 인코딩 형태의 캐릭터 타입이다. `TCHAR` 로 반환하는 오퍼레이터 변환을 오버로딩해서 생 데이터에 접근할 수도 있다고 한다.
  * 이 외에도 `TChar<TCHAR> == FChar` 도 있는데 이것은 각각의 `TChar` 을 사용할 수 있도록 도와주는 함수를 제공한다.

## Container

`UE4` 에서 자체로 지원하는 컨테이너는 여러가지가 있지만 그 중에서 가장 많이 사용되는 것은 

1. `TArray`
2. `TMap`
3. `TSet`

이다.

### `TArray`

`std::vector<>` 와 비슷하지만 좀 더 많은 기능을 제공한다고 한다. 내부에서 `UPROPERTY` 을 사용하기 때문에 리플렉션 및 `GC` 도 대응을 한다. 그리고 대개 타입은 `UObject*` 에서 파생되는 타입들이 들어간다고 한다...

> 자세한 문서는 [여기](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/TArrays) 를 참조한다.

### `TMap`

`std::map` 과 비슷하지만 좀 더 기능이 많다. 키로는 `GetTypeHash()` 함수를 구현하고 있는 어떠한 타입이라도 가능하다. 

> 자세한 문서는 [여기](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/TMap) 를 참조한다.

### `TSet`

`std::set` 과 비슷하지만 좀 더 빠르고 성능이 좋다고 한다. 사실 `TArrays<>` 도 `AddUnique` 와 `Contains` 을 사용해서 `TSet` 을 구현할 수 있으나 `TSet` 이 성능면에서 더 좋다.

* 중요한 것은 `UE4` 의 `Reflection` 과 `GC` 을 이용하기 위해서는 `TArray<>` 만 써야한다. 다른 컨테이너 타입들은 지원을 하지 않기 때문에 힙 할당된 객체가 뜬금없이 `GC` 가 되거나 하는 수가 있다.

### Container 이터레이터

이터레이터를 사용해서 컨테이너의 요소들에 접근할 수 있다.

{% highlight c++ %}
// Start at the beginning of the set, and iterate to the end of the set
for (auto EnemyIterator = EnemySet.CreateIterator(); EnemyIterator; 
     ++EnemyIterator) {
 AEnemy* Enemy = *EnemyIterator;
 if (Enemy.Health == 0) {
   // 'RemoveCurrent' is supported by TSets and TMaps
   EnemyIterator.RemoveCurrent();
 }
}
{% endhighlight %}

### Using for-each loop

`TArray` 와 `TSet` 그리고 `TMap` 역시 C++11 부터 제공되는 For-each loop 을 사용할 수 있다.

### `TSet` 및 `TMap` 에서 해쉬 펑션 만들기

{% highlight c++ %}
class FMyClass {
    uint32 ExampleProperty1;
    uint32 ExampleProperty2;

    // Hash Function
    friend uint32 GetTypeHash(const FMyClass& MyClass) {
        // HashCombine is a utility function for combining two hash values
        uint32 HashCode = HashCombine(MyClass.ExampleProperty1, 
                                      MyClass.ExampleProperty2);
        return HashCode;
    }

    // For demonstration purposes, two objects that are equal
    // should always return the same hash code.
    bool operator==(const FMyClass& LHS, const FMyClass& RHS) {
        return LHS.ExampleProperty1 == RHS.ExampleProperty1
            && LHS.ExampleProperty2 == RHS.ExampleProperty2;
    }
};
{% endhighlight %}

