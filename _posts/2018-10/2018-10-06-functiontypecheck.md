---
layout: single
title: "Templateを使用したコンパイル時の可変長型引数マッチング (2)"
tags: C++
date: 2018-10-06 +0900
categories: C++
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

こんにちは、おとといの[Templateを使用したコンパイル時の可変長型引数マッチング]()からの続編となります。
今日はまた個人プロジェクトをやる時間を借りて**関数の引数の型**をとってそれを任意型引数とマッチングする奇妙なやつを作りました。

元々、プロジェクトなどに適用としたかったのがこの*関数の引数型と可変長型引数のタイプマッチング*でしたのでこれで目標は達成です。

## 参照

> [Variadic variadic template templates](https://stackoverflow.com/questions/9662632/variadic-variadic-template-templates)

やはりStackoverflowは離せない僕らの友達ですわ。ものすごく参考になりました。

## コード全文

``` c++
namespace tmp
{

// Use this function to check TArgs is able to be inputted as TGoal.
template <typename TGoal, typename TArg>
[[nodiscard]] constexpr bool __TryMatchGoalArg() noexcept
{
    using _TGoal = std::remove_reference_t<std::remove_cv_t<TGoal>>;
    using _TArg  = std::remove_reference_t<std::remove_cv_t<TArg>>;
    
    constexpr bool _typeMatched = std::is_same_v<_TGoal, _TArg> || std::is_constructible_v<_TGoal, _TArg>;
    return _typeMatched;
};

// Specify function type parameters.
template <typename...> struct _Types {};

// Use this type to extracts parameter types from function. (stores to _Types<...>!)
template <typename TSig> 
struct _Args;
template <typename TRet, typename... TArgs>
struct _Args<TRet (*)(TArgs...)> { using ParamType = _Types<TArgs...>; };
template <typename TType, typename TRet, typename... TArgs>
struct _Args<TRet (TType::*)(TArgs...)> { using ParamType = _Type<TArgs...>; };

// Use this type to execute function param type matching test.
template <typename...> struct _TestFunc;
template <template <typename... TV> class TCont, typename... TU, typename... TV>
struct _TestFunc<TCont<TV...>, TU...> 
{
    // Integrity test
    static_assert(std::is_same_v<_Types<TV...>, TCont<TV...>>, "");
    
    // Parameter count checking test
    static constexpr bool _countMatched = sizeof...(TV) == sizeof...(TU);
    static_assert(_countMatched, "Test failed, type parameter counts are not matched.");

    // Type matching test
    template <bool flag = (... && __TryMatchGoalArg<TV, TU>())>
    struct Result : public std::integral_constant<bool, flag> {};
};

} /// ::tmp namespace

// HELPER MACRO 
#define M_TEST_MATCH_FUNCTIONPARAM(__MAErrorString__, __MAFunction__, ...) \
    static_assert(tmp::_TestFunc<tmp::_Args<decltype(__MAFunction__)>::ParamType, __VA_ARGS__> \
        ::Result<>::value, __MAErrorString__)

// HELPER FUNCTION 
template <typename TGoalFunction, typename... TArgs> 
constexpr void TestMatchingFunctionParamTypes(TArgs&&...)
{
    using TGoalType = typename tmp::_Args<TGoalFunction>::ParamType;
    static_assert(tmp::_TestFunc<TGoalType, TArgs...>::template Result<>::value, "");
}

template <typename TGoalFunction, typename... TArgs> 
constexpr void TestMatchingFunctionParamTypes()
{
    using TGoalType = typename Args<TGoalFunction>::ParamType;
    static_assert(TestFunc<TGoalType, TArgs...>::template Result<>::value, "");
}
```

## 変更点

* 前の `Test<>::TypeMatched<>::...` とは違って`_TestFUnc<>`::構造体一つ使うことで終わらせるようにしました。(厳密にいいますと`Result`まで２つですが)
* 関数の引数タイプを検証するときには基本関数ポインターを使って渡すようにしました。統一性を保つためです。

## 使用例

``` c++
int  Function(int, const std::string&);
void Function2(const std::string&, const std::string_view&, float, double);
void Function3(const std::string&, const std::vector<int>&, long long, const int*);

struct DTest
{
    void Function(const int, const std::string&);
};

int main()
{
    std::string i = "Hello world";
    std::vector<int> vec = {1, 2, 3, 4, 5};
    int a = 362'555;
    const char* hel = "Hell";
    int* hA = new int(450);
    
    using namespace tmp;

    static_assert(_TestFunc<_Args<decltype(&Function)>::ParamType, int, std::string&>::Result<>::value, "");
    static_assert(_TestFunc<_Args<decltype(&DTest::Function)>::ParamType, int, const char*>::Result<>::value, "");
    
    M_TEST_MATCH_FUNCTIONPARAM("FAILED!", &Function, int, std::string&);
    M_TEST_MATCH_FUNCTIONPARAM("FAILED!", &Function2, const char*, const char*, float, float);
    M_TEST_MATCH_FUNCTIONPARAM("FAILED!", &DTest::Function, char, std::string);

    TestMatchingFunctionParamTypes<decltype(&Function3)>(i, vec, a, hA);
    TestMatchingFunctionParamTypes<decltype(&DTest::Function)>(a, hel);
    TestMatchingFunctionParamTypes<decltype(&DTest::Function), int16_t, const char*>();
    
    delete hA;
}
```