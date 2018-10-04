---
layout: single
title: "例外処理の3種類分析"
tags: C++
date: 2018-09-28 +0900
categories: C++
comments: true
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# 例外処理の3種類方法

## 1. ウィンドウズだけの`__try``__except`

Windows系列(NTを除く)のOSで提供する例外処理方法である。一般的にSEH（Structured Exception Handling）と呼ばれる。
既存のC++例外とはちょっと違って、アクセス違反によるランタイムエラーも処理が可能だそうだ。すなわちランタイムにエラーによってプログラムを終了しなければならないイベントが発生した時に、それをランタイムで処理するようにしている。

``` c++
VOID TryExcept()    
{    
  __try     
  {    
    std::wprintf(L"__try!!\n");    
    *(PINT)NULL = 0;  
  } 
  __except(EXCEPTION_EXECUTE_HANDLER)    
  {    
    std::wprintf(L"__except!!\n");    
  }    

  std::wprintf(L"Finish Function!\n");    
}
```

しかしそのSEHの`__try`文でC++のデストラクタがついてあるインスタンスを使おうとしたら別のオプションを付けて使わなければならない。オプションをしなかった場合にはデストラクタが呼び出されないままに例外処理が行われる。デストラクタを呼び出すためには`/EH`オプションをつけるべき。

`__except`フィルターに付けられる例外処理の仕方は以下のように3つがある。

* `EXCEPTION_CONTINUE_EXECUTION` 例外を無視する。例外が発生したらそのままずっとコードを実行する。 
* `EXCEPTION_CONTINUE_SEARCH` 例外を認識せず、上位のSEH構文を探して例外を投げる。その間にスタックは解除される。
* `EXCEPTION_EXECUTE_HANDLER` 例外を認識して処理する。

## 2. 普通の`try``except`



## 3. Unexpected Exception Filter



# 参照

> [http://kuaaan.tistory.com/435](http://kuaaan.tistory.com/435)

> [https://msdn.microsoft.com/ko-kr/library/swezty51.aspx](https://msdn.microsoft.com/ko-kr/library/swezty51.aspx)

> [https://msdn.microsoft.com/ko-kr/library/s58ftw19.aspx](https://msdn.microsoft.com/ko-kr/library/s58ftw19.aspx)

