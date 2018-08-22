---
layout: single
title: "CRTP Memory-Locality Pattern"
tags: C++ Design-Pattern
date: 2018-08-22 +0900
categories: C++
comments: true
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

## Reference

> [OutOfLine - A Memory-Locality Pattern for High Performance C++](https://blog.headlandstech.com/2018/08/15/outofline-a-memory-locality-pattern-for-high-performance-c/)

を参考にしました。

## Cache

ほぼ全てのプラットフォームでほぼ全てのアプリケーションはCPUにプロセッサーを適材しながら主メモリなどから（RAM）処理に必要なメモリなどに接近して処理したりします。しかし情報学を専攻していらっしゃった方々には当然ご存知のように、CPUのレジスタとRAMのメモリ空間の処理速度にはとんでもない速度差があります。

それでその速度やメモリ空間の差を詰めるためにできたのが[Cache memory](https://en.wikipedia.org/wiki/CPU_cache)ですね。それによって処理するメモリ領域に対し、ある程度の周辺のメモリ領域までキャッシュに適材してMemoryまで行くことなくもっと素早く処理を行うことができます。

ですが問題はまだ残っています。もしかしてあるメモリの領域にデータのリストがあったとしてそしてそのデータには色んなデータがインスタンスなどで内包されています。ですがCPUではそのデータインスタンスの中である一部だけを数多く処理したがっています。しかしそのデータを処理するためにはまずCacheに載せることが必要となっていますしそして各インスタンスのバイト容量は一部のデータに接近するに比べて大きすぎます。

それじゃ幾らCacheが大きくてもMissによって処理時間はあんまり縮めないと思います。（それにCacheもメモリですので大きくなると接近速度が下がります。）僕はデータクラスの構造を保ちながらCacheのLocalityを十分活用して一部のデータだけをはやく接近されるようにしたいと思います。

## Hot Data? Cold Data?

> [Hot Data](https://www.techopedia.com/definition/32621/hot-data)
> [Cold Data](https://www.techopedia.com/definition/32623/cold-data)

`Hot Data`は頻繁に接近して、処理を行うデータを指します。逆に`Cold Data`はあんまり接近せずにいているデータを指しますね。上の例に例えるとCPUが接近する一部のデータは`Hot Data`を、そしてその他のデータは`Cold Data`を指します。

## Example Code

``` c++
#include <cstdint>
#include <chrono>
#include <iostream>
#include <string>
#include <unordered_map>
#include <memory>
#include <random>

template <typename THotData, typename TColdData>
class IColdData {
public:
  template <typename... TArgs>
  IColdData(TArgs&&... args) : IColdData(DMapInit{}) {
    pInitializeColdData(std::forward<TArgs>(args)...);
  }
  ~IColdData() {
    if (sGlobalMap.find(this) != sGlobalMap.end()) { sGlobalMap.erase(this); }
  }

  explicit IColdData(IColdData&& moveObj) noexcept : IColdData(DMapInit{}) {
    (*this) = std::move(moveObj);
  }

  IColdData& operator=(IColdData&& moveObj) noexcept {
    sGlobalMap[this] = std::move(sGlobalMap[&moveObj]);
    sGlobalMap.erase(&moveObj);
    return *this;
  }

  IColdData(const IColdData&) = delete;
  IColdData& operator=(const IColdData&) = delete;

  TColdData& GetCold() noexcept { return *sGlobalMap[this]; }
  const TColdData& GetCold() const noexcept { return *sGlobalMap[this]; }

private:
  inline static std::unordered_map<IColdData const*, std::unique_ptr<TColdData>> sGlobalMap;

  struct DMapInit final {};
  IColdData(DMapInit) { sGlobalMap.try_emplace(this); };

  template <typename... TArgs>
  void pInitializeColdData(TArgs&&... args) {
    sGlobalMap.find(this)->second = std::make_unique<TColdData>(std::forward<TArgs>(args)...);
  }

  void pReleaseColdData() {
    sGlobalMap[this].reset();
  }
};

struct DSlow final 
{
    std::string j;
    std::int32_t i;
};

struct DFast final : public IColdData<DFast, std::string>
{
    std::int32_t i;
};

struct DFastest final
{
    std::int32_t i;
};

//
// We time to measure throughput of touching all the fast data once in sequence
//
template <class Data>
std::pair<unsigned long long, std::int32_t> Trial(const Data& data) {
    std::int32_t result = 0U;
    const auto before = std::chrono::system_clock::now();
    for (const auto& d : data) { result += d.i; }
    const auto after = std::chrono::system_clock::now();

    using std::chrono::duration_cast;
    using std::chrono::microseconds;
    return std::make_pair(duration_cast<microseconds>(after - before).count(), result);
}

//
// We synthesize the data up-front so the generation 
// can't interfere with the measured behavior in any way
//
template <class Data>
auto MakeData() {
  std::mt19937 rng(std::chrono::system_clock::now().time_since_epoch().count());
  std::vector<Data> data(10000000);
  for (Data& d : data) { d.i = rng(); }
  return data;
}

template <typename TData>
void Test(const char* const name) 
{
    unsigned long long _time = 0;
    unsigned long long value = 0;
    std::int32_t loop = 10;
    
    const auto list = MakeData<TData>();
    for (int i = 0; i < loop; ++i)
    {
        auto [microTime, resultValue] = Trial(list);
        _time += microTime;
        value += resultValue;
    }
    
    std::cout << name << " took " << _time / loop << "us and " << value << '\n';
}

int main()
{
    Test<DSlow>   ("With cold data in-line (orignal) ");
    Test<DFast>   ("With IColdData                   ");
    Test<DFastest>("With only hot data               ");
}
```

``` text
g++ -std=c++17 -O2 -Wall -Werror -pedantic -pthread main.cpp && ./a.out
With cold data in-line (orignal)  took 61620us and 1186974040
With IColdData                    took 12738us and 8979678320
With only hot data                took 13102us and 1906102120
```

ご覧に通りに普通のコンテナーよりも`IColdData`を使ったCRTPコンテナーが5倍くらい速いことがわかります。なぜ`Hot data`だけあるデータより処理時間が短いのかは分かりませんが誤差だと思いましょうね。