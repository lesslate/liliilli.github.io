---
layout: single
title: "CRTPとMagic Staticを用いてSingletonを作る"
tags: C++ Design-Pattern
date: 2018-08-21 +0900
categories: C++
comments: true
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

GoFから伝えられて来た、今でもあるところには使っている筈のパターんである[Singletonパターン](https://en.wikipedia.org/wiki/Singleton_pattern)は容易くデータが管理できる利点があります。もちろん、ユニットテストがちゃんと通らない仕様であってそして昔にはスレット安全ではないという致命的な短所もありますね。（それに関しての論争もまだ続いているようですが加担したくはないんですね。）

だけどC++11からは関数の中に`Static`をつけるとその`Static`のデータはMagic StaticというスレッドセーフにStaticインスタンスが生成される仕様があってそれを用いて簡単にスレッド安全性は確保できます。（但し、返還されたシングルトンに同時に接近するのは又別の問題。）その仕様はMSVCでは2015から実装されましたが、ちょっと動作が他のものと違うようですので2017をおすすめします。

とにかく昔だったら`Mutex`を二重でかけて接近したり、その分で処理速度は落ちたりしてて使用することを躊躇したかもしれなかったけど今はそれがなくなったため、個人的には積極的に使用しています。もちろんユニットテストのことを考慮しながら作成します。

## CRTP (Curiously Recurring Template Pattern) とは

> [Curiously Recurring Template Pattern](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern)

`CRTP`という`C++`の特有のパターンは継承するクラステンプレートの型タイプに自分の型を使ってテンプレートクラスを生成するIdiomです。これによって普段使っていた`Virtual`を活用した`動的多型を回避して（Vtableによる間接的な呼び出しが省略できる）静的多型が実現できます。

もちろん、特殊すぎるパターンですので実開発ではあんまり使われる機会がないようですが、`Singleton`はこれを積極的に活用して実装することができるようです。なぜなら`Singleton`は`static`を活用するからですね。

## Example Code 

それで実装しました。
次は今の個人プロジェクトに使われているCRTPを用いた`Singleton`パターンの例です。

C++17を前提としておりますが、属性やらを除けば最小C++11以上で使えることができます。

``` c++
#define MDY_SINGLETON_PROPERTIES(__MASingletonType__) \
public: \
    __MASingletonType__(const __MASingletonType__##&) = delete; \
    __MASingletonType__(__MASingletonType__##&&) = delete; \
    __MASingletonType__##& operator=(const __MASingletonType__##&) = delete; \
    __MASingletonType__##& operator=(__MASingletonType__##&&) = delete

#define MDY_SINGLETON_DERIVED(__MADerivedSingletonType__) \
private:                                                  \
    __MADerivedSingletonType__() = default;               \
    virtual ~__MADerivedSingletonType__() = default;      \
    [[nodiscard]] EDySuccess pfInitialize();              \
    [[nodiscard]] EDySuccess pfRelease();                 \
    friend class ISingleton<__MADerivedSingletonType__>

template <typename TType>
class ISingleton
{
public:
  inline static TType& GetInstance() noexcept
  {
    static TType instance;
    return instance;
  }

  [[nodiscard]]
  inline EDySuccess static Initialize() noexcept
  {
    return GetInstance().pfInitialize();
  }

  [[nodiscard]]
  inline EDySuccess static Release() noexcept
  {
    return GetInstance().pfRelease();
  }

protected:
  ISingleton() = default;
  virtual ~ISingleton() = default;

  MDY_SINGLETON_PROPERTIES(ISingleton);
};

class MDyTime final : public ISingleton<MDyTime>
{
  MDY_SINGLETON_DERIVED(MDyTime);
  MDY_SINGLETON_PROPERTIES(MDyTime);
public:
  int GetI() const noexcept
  {
  	return this->i;
  }

private:
  int i = 13337;
};

int main() 
{
    auto& type = MDyTime::GetInstance();
    return type.GetI();
}
```

