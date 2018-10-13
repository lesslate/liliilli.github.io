---
layout: single
title: "Combination Exhausitive Search"
tags: Algorithm
date: 2018-10-11 +0900
categories: Algorithm
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

``` c++
std::vector<char> Setup()
{ 
    using namespace neu::test;
    
    std::array<char, 'Z'- 'A' + 1> alphabets;
    for (int32_t i = 0; i < ('Z' - 'A' + 1); ++i)
    {
        alphabets[i] = 'A' + i;
    }
    
    static std::random_device rd;
    static std::mt19937 g(rd());
    std::shuffle(alphabets.begin(), alphabets.end(), g);
    
    const int32_t numbers = 10;
    
    std::vector<char> container{};
    container.reserve(numbers);
    for (int32_t i = 0; i < numbers; ++i) { container.emplace_back(alphabets[i]); }
    
    // Print informations
    std::printf("|");
    for (const auto& value : container)
    {
        std::printf("%c|", value);
    }   std::printf("\n");
    
    return container;
}

void __PrintOutput(_MIN_ const std::vector<char>& output)
{
    for (const auto& chr : output)
    {
        std::printf("%c", chr);
    }   std::printf("\n");
}

void __ProcessPermutation(_MIN_ const std::vector<char>& input, _MIN_ const int32_t cursorId,
                          _MIN_ const int32_t level, _MOUT_ std::vector<char>& output)
{
    if (level == 0) { __PrintOutput(output); return; }
    else 
    {
        const int32_t size = static_cast<int32_t>(input.size());
        for (int32_t i = cursorId + 1; i < size; ++i)
        {
            output.emplace_back(input[i]);
            __ProcessPermutation(input, i, level - 1, output);
            output.pop_back();
        }
    }
}

void ProcPermutationExhaustiveSearch(_MIN_ std::vector<char>& input)
{ 
    const int32_t size     = input.size();
    std::vector<char> list = {};
    
    for (int32_t i = 1; i <= size; ++i)
    {
        __ProcessPermutation(input, -1, i, list);
    }
}

int main()
{
    neu::test::Execute(&Setup, &ProcPermutationExhaustiveSearch, "Test", 50);
}
```