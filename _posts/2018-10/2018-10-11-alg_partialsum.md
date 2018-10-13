---
layout: single
title: "Partial Maximum Sum"
tags: Algorithm
date: 2018-10-11 +0900
categories: Algorithm
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

``` c++
#include <cstdio>
#include <cstdint>
#include <cassert>

#include <chrono>
#include <vector>
#include <random>

#define _MIN_    // Input parameter
#define _MOUT_   // Output parameter
#define _MINOUT_ // Input & Output parameter
 
// @link https://gist.github.com/liliilli/59ab5d195d12f060b55c5eb1b6e5ce55

std::vector<std::int32_t> Setup()
{ 
    using namespace neu::test;
    std::vector<std::int32_t> container{};
    
    constexpr int32_t count = 10000;
    container.reserve(count);
    
    for (int32_t i = 0; i < count; ++i) { container.emplace_back(GetRandomInteger<int32_t>(-100, 50)); }
    
    return container;
}

[[nodiscard]]
int32_t GetMaximumSum(_MIN_ const std::vector<std::int32_t>& input, 
                      _MIN_ const int32_t start, _MIN_ const int32_t end)
{
    assert (start <= end);
    if (start == end) { return input[start]; }
    
    const int32_t mid   = (start + end) / 2;
    int32_t leftMaxSum  = std::numeric_limits<int32_t>::min();
    int32_t rightMaxSum = std::numeric_limits<int32_t>::min();
    int32_t presentSum  = 0;
    
    for (int32_t i = mid; i >= start; --i)
    {
        presentSum += input[i];
        leftMaxSum = std::max(leftMaxSum, presentSum);
    }
    
    presentSum = 0;
    
    for (int32_t i = mid + 1; i <= end; ++i)
    {
        presentSum += input[i];
        rightMaxSum = std::max(rightMaxSum, presentSum);
    }
    
    const int32_t childMaxSum = std::max(GetMaximumSum(input, start, mid), GetMaximumSum(input, mid + 1, end));
    return std::max(childMaxSum, leftMaxSum + rightMaxSum);
}

void ProcPartialSetMaximumSum(_MIN_ std::vector<std::int32_t>& input)
{ 
    const int32_t start = 0;
    const int32_t end   = input.size() - 1;   
    
    const int32_t value = GetMaximumSum(input, start, end);
    std::printf("Maximum sum value : %3d\n", value);
}

int main()
{
    neu::test::Execute(&Setup, &ProcPartialSetMaximumSum, "PartialMaximumSum", 50);
}
```