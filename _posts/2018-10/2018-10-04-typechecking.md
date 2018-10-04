---
layout: single
title: "Templateを使用したコンパイル時の可変長型引数マッチング"
tags: C++
date: 2018-10-04 +0900
categories: C++
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
## まずなんで作りたかったのか

`C++`で個人プロジェクトの開発をしている途中、`template`機能やその中身で`if constexpr`、`type_traits`、そして任意関数に可変長引数を`std::forward<>`として渡さねければいけない関数を作らなければいけないときがありました。

`if constexpr` `type_traits`を用いたコンパイル時の型引数の分析はどうでも良いですが、任意関数に可変長引数を渡して適切に関数の呼び出しを行うことが一番心配でした。なぜなら`std::forward<>`で可変長引数を渡すことが失敗になると、STLの特有の難読化とまざってエラーメッセージがものすごく酷くなるからです。

そのため予め可変長引数の型たちが呼び出そうとする関数に変換などを通して渡せるかを確認するコンパイル時実行関数を作らなければなりませんでした。関数の実装には`C++17`を積極的に使用しています。

## コード全文

``` c++
template <typename TGoal, typename TArg>
[[nodiscard]] constexpr bool __TryMatchGoalArg() noexcept
{
    using _TGoal = std::remove_reference_t<std::remove_cv_t<TGoal>>;
    using _TArg  = std::remove_reference_t<std::remove_cv_t<TArg>>;
    
    return std::is_same_v<_TGoal, _TArg> || std::is_constructible_v<_TGoal, _TArg>;
};

template <typename... T> 
struct Test
{
    template <typename... U>
    struct TypeMatched
    {
        enum {
            TGoalTypesCount = sizeof...(T),
            TArgsTypesCount = sizeof...(U)
        };
        
        static constexpr bool _countMatched = TGoalTypesCount == TArgsTypesCount;
        static_assert(_countMatched, "Test failed, type parameter counts are not matched.");

        template <bool flag = (... && __TryMatchGoalArg<T, U>())>
        struct Result : public std::integral_constant<bool, flag> {};
    };
};
```

## 分析したいところ

### 1. 二重の可変長型引数を受け入れるためにはあれしか方策がなかったか

``` c++
template <typename... T> 
struct Test
{
    template <typename... U>
    struct TypeMatched
    {
```

`C++`のTMPを愛している他のユーザーたちが書いたものでは`struct`一つで二重を受け入れられる方法がありうるです。ですがまだ自分のTMPコードを書く能力が足りないため、結局`struct`を二重で書く方法で可変長を二重で受けるように書きました。スイマセン

### 2. なぜ`enum`を使った？

``` c++
enum {
    TGoalTypesCount = sizeof...(T),
    TArgsTypesCount = sizeof...(U)
};
```

`enum` `enum class`の列挙型はコンパイル時に生成される厳密にいいますと定数です。そしてテンプレートがある`struct`の中にある定数、`static`定数は特殊化されて具体化したコードによってそれぞれ近います。

これを用いて渡される型引数や長さによってそれぞれ違う`enum`定数が生成されるようにしました。`TGoalTypesCount` `TArgsTypesCount`は本テストに入る前に型引数の長さが一致しているかを確かめます。

### 3. `C++17`の新機能、畳み込み式

> [畳み込み式](https://cpprefjp.github.io/lang/cpp17/folding_expressions.html)
> ...畳み込み式 (fold expression) は可変引数テンプレートのパラメータパックに対して二項演算を累積的に行う (畳み込む fold)。...

``` c++
template <bool flag = (... && __TryMatchGoalArg<T, U>())>
struct Result : public std::integral_constant<bool, flag> {};
```

畳み込み式がすごいよとはインターネットなどでよく言われてきましたが、実際使ったのは今のコードを実装しながらが初めてでした。果たしてコンパイル時に型引数に対して畳み込みができるか疑っていましたがきちんとできます。うれしいね！

``` c++
(... && __TryMatchGoalArg<T, U>())
```

上の畳み込み式を解きほぐしますと以下のようになります。

``` c++
/// __TryMatchGoalArgは__Tryに略します。
(((__Try<T0, U0>() && __Try<T1, U1>()) && _Try<T2, U2>()) ... && _Try<TN, UN>())
```

### 4. `constexpr` タイプ比較関数

じつは`lambda`式に型引数を渡したかったですが、まだ支援していないようですので速やかにあきらめて`constexpr`の関数を作成しました。

``` c++
using _TGoal = std::remove_reference_t<std::remove_cv_t<TGoal>>;
using _TArg  = std::remove_reference_t<std::remove_cv_t<TArg>>;
```

注意すべき点は`std::decay_t<>`を使わずに一々`cv-qualifier`と`reference`を取り除いているところです。なぜなら`std::decay_t<>`をしますと`const char*`を渡す場合に`char`になってしまうため、もし`const char*`が`std::string`に変換または生成ができるかを訪ねたら失敗になります。それを事前に防ぐために`std::decay<>`は使いません。

>ちなみに`C++20`からは`cv`と`reference`だけを取り除くようにする`type_traits`構造体も追加されるそうですが、搭載にはまだ遠いですね。

## 使い道

このように使います。ちょっとひどいですがご了承ください。

``` c++
static_assert(Test<int, int>::TypeMatched<int, int>::Result<>::value, "TEST FAILED");
static_assert(Test<int, float, double, char, const std::string>::
              TypeMatched<int, float, double, char, const std::string&>::
              Result<>::value, "TEST FAILED");

template <typename... TGoalTypes, typename... TArgs>
void TestFunction(TArgs&&...)
{
    static_assert(Test<TGoalTypes...>::
                  template TypeMatched<TArgs...>::
                  template Result<>::value, "Test failed");
}

int main()
{
    std::string i = "Hello world";
    std::vector<int> vec = {1, 2, 3, 4, 5};
    int a = 362'555;

    int* hA = new int(450);

    TestFunction<const std::vector<int>&, const std::string&, const int, const int*>
        (vec, i, a, hA);

    delete hA;
}
```

