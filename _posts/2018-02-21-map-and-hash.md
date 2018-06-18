---
layout: post
title: map 과 unordered_map차이
tags: C++
categories: C++
comments: false
mathjax: true
date: 2018-02-21 11:46:03
---


현재 내가 만들고 있는 [OPGS16](https://github.com/liliilli/OPGS16) 은 오브젝트, 쉐이더 및 텍스쳐에 접근해야 할 때 해당 자원의 "이름" 을 통해서 접근하는 것이 가능하도록 되어있다. 이 때 자주 사용했던 자료구조가 [**Hash table**](https://en.wikipedia.org/wiki/Hash_table) 이었고, 이를 C++ 에서 구현한 것이 C++11 부터 지원되기 시작한 *unordered_map* 이다.
<!-- more -->
사실 C++11 이전에도 키와 값을 가지고 해쉬 아닌 해쉬 맵을 구현한 것이 있었는데 그것이 *map* 이었다. 문제는 이 *map* 이 실제 해쉬 함수를 사용한 것도 아니기 때문에 해쉬의 특성을 반영하고 있지 않았다는 점이었다. *map* 은 [**Red-black tree**](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree) 라는, 밸런싱 이진 트리를 사용해서 구현되었기 때문에 $ O(lg n) $ 의 탐색, 삽입, 제거 성능을 띈다는 점이 있다. $ O(1) ~ O(n) $ 까지의 성능을 보이는 해쉬 맵과는 전혀 딴판이다.

아무튼 나는 OPGS16 의 키, 버켓 자료구조로 unordered_map 을 쓰기로 했는데, 개발을 하다보니까 살짝 unordered_map 과 map 의 성능 차이에 의문을 품게 되었다. 과연 이게 항상 빠른걸까? 이론상으로는 해쉬의 탐색 성능이 최적 $ O(1) $ 이지만 해쉬 함수를 거치는 비용과 miss 될 때의 비용은?... 그래서 한번 간단하게 map 과 unordered_map 의 성능 차이를 테스트 해보기로 했다.

그리고 지금까지는 키를 항상 *std::string* 으로 받고 있었지만 스택오버플로우에서 const char* 을 키로 쓰는 것이 더 빠르다는 글을 찾게 되었기 때문에 그것 역시 한번 비교를 해보고 싶었다. 

## map, unordered_map

우선 lorem ipsum 단어 생성기 [사이트](https://www.lipsum.com/)에서 100 개의 단어를 골라, 각 단어를 토큰화한 뒤에 unordered_map 과 map 의 컨테이너에 삽입, 참조 (find() 을 사용), 제거를 3000번 루프해봤다.

코드는 다음과 같았다.

``` c++
#include <algorithm>/*! std::algorithm */
#include <iostream> /*! std::cout */
#include <list>     /*! std::list */
#include <string>   /*! std::string */
#include <utility>  /*! std::hash */
#include <sstream>  /*! std::stringstream */
#include <chrono>   /*! std::chrono */

#include <type_traits>
#include <map>      /*! std::map using RBtree */
#include <unordered_map>    /*! std::unordered_map */

constexpr auto COUNT = 5'000;

const std::string _str = 
"Lorem ipsum dolor sit amet consectetur adipiscing elit Quisque eget vulputate diam Maecenas sit amet nulla a lectus aliquet facilisis at eget augue Praesent a arcu a quam porta convallis nec sed elit\n"
"Vivamus semper diam augue ac mattis sapien porttitor sed Aenean et felis mattis augue porta venenatis Duis fermentum volutpat urna at mattis lectus faucibus pellentesque Cras mollis condimentum turpis id elementum\n"
"Integer aliquam dolor at ex feugiat quis pellentesque ipsum pretium Integer venenatis felis id dui euismod in convallis ante sollicitudin Morbi nunc libero iaculis sit amet orci at aliquet sagittis dui\n"
"Sed et quam dictum commodo quam ac gravida nisl Nullam ut eros sagittis pharetra tortor vitae elementum tellus Duis eu tristique augue in laoreet justo Integer hendrerit massa at ligula tempor placerat\n"
"Duis tristique nisl vel scelerisque luctus Aliquam a felis eu ex elementum efficitur Quisque accumsan velit sit amet dignissim ultricies Phasellus turpis eros tincidunt ac velit eu rutrum feugiat sapien\n"
"Cras ullamcorper pellentesque mauris non dictum Ut turpis nibh, aliquam in consectetur vitae congue in odio Integer lacinia lorem id tortor convallis id euismod metus sodales Phasellus at malesuada nunc\n"
"Curabitur ut tincidunt nunc Duis risus risus laoreet luctus ligula sed tincidunt lacinia";

template <class _Ty>
void Process(_Ty& hash_map, const std::list<const char*>& token_list) {
    for (auto i = 0; i < COUNT; ++i) {
        /*! Insertion */
        for (const auto& tkn : token_list)  
            hash_map[tkn] = std::hash<const char*>{}(tkn);
        /*! Find */
        std::for_each(token_list.rbegin(), token_list.rend(), [&hash_map](const auto& str) {
            hash_map.find(str);
        });
        /*! Clear */
        for (const auto& tkn : token_list)
            hash_map.erase(tkn);
    }
}

template <class _Ty>
std::chrono::duration<double> Count(_Ty& container, const std::list<const char*>& list) {
    auto start = std::chrono::system_clock::now();
    Process(container, list);
    auto end = std::chrono::system_clock::now();
    
    return end-start;
}

int main() {
    std::ios_base::sync_with_stdio(false);
    
    std::unordered_map<const char*, int> cchar_hash;
    std::unordered_map<std::string, int> string_hash;
    std::map<const char*, int>          cchar_map;
    std::map<std::string, int>          string_map;
    
    std::stringstream stream{_str};
    std::list<const char*> token_list;
    
    std::string token;
    while (stream >> token) {
        if (!token.empty()) token_list.emplace_back(token.c_str());
    }
    
    auto string_diff = Count(string_hash, token_list);  /*! std::string hash */
    auto cchar_diff  = Count(cchar_hash, token_list);    /*! const char* hash */
    auto s_map_diff  = Count(string_map, token_list);    /*! std::string map */
    auto c_map_diff  = Count(cchar_map, token_list);    /*! std::string map */
    
    std::cout << "std::string hash for 100 words loop 3000 times : " << string_diff.count() << "s\n";
    std::cout << "const char* hash for 100 words loop 3000 times : " << cchar_diff.count() << "s\n";
    std::cout << "std::string  map for 100 words loop 3000 times : " << s_map_diff.count() << "s\n";
    std::cout << "const char*  map for 100 words loop 3000 times : " << c_map_diff.count() << "s\n";
    return 0;
}
```

왜 긴 단어가 아니라 작은 단어들을 표본으로 삼았냐면, OPGS16 에서는 긴 단어의 키가 쓰일 일이 잘 없기 떄문이다. 물론 프레임워크를 쓰는 유저가 긴 단어를 쓰겠다고 하면 문제가 되겠지만...

결과는 다음과 같았다.

``` bash
g++ -std=c++17 -O1 -Wall -pedantic -pthread main.cpp && ./a.out
std::string hash for 100 words loop 3000 times : 0.193673s
const char* hash for 100 words loop 3000 times : 0.0460765s
std::string  map for 100 words loop 3000 times : 0.11217s
const char*  map for 100 words loop 3000 times : 0.0149009s
```

어찌 된 일인가 unordered_map 보다 map 이 더 빠른 결과가 나왔다. 특히나 최적화 옵션을 붙이지 않고 쓰면 unordered_map 은 0.7초, map 은 0.3초가 걸리는 것을 알았는데 이는 좀 한방 먹은 느낌이었다. 

이는 인터넷에서도 찾은 다른 분의 글에서도 찾을 수 있었는데, 문자열을 키로 설정했을 경우에는 문자열의 크기에 따라 unordered_map 과 map 의 실행 성능 교차점이 미뤄지는 것을 확인할 수 있었다. [글 링크](http://veblush.blogspot.kr/2012/10/map-vs-unorderedmap-for-string-key.html)

### 시간 복잡도

위 글에 따르면, 그리고 RB-tree 와 Hash table 의 성능에 따르면 문자열 탐색의 성능은 다음과 같아야 한다.
$$
\text{map 에서의 문자열 비교} = O(lgN) \\\
\text{map 에서 마지막 문자열과의 비교} = O(1) \\\
\text{hash_map 의 Hash function} = O(1) \\\
\text{hash_map 의 키 비교} = O(1)
$$
그런데 문제는 문자열의 경우에는 "문자 단위" 탐색 작업의 시간 복잡도를 고려해봐야 된다는 트랩이 있다.
$$
\text{map 에서의 노드마다 각 문자 비교} = unknown \\\
\text{map 에서 마지막 문자열의 각 문자 비교} = O(L) \\\
\text{hash_map 의 Hash function} = O(L) \\\
\text{hash_map 의 키 비교} = O(L)
$$
여기서 레드-블랙 트리가 완벽하게 분할되었다고 가정하면, $ L $ 길이를 가지는 문자열 키를 탐색할 때 각 깊이 $ d $ 에 대해 문자열의 각 위치의 문자 혹은 고정된 $ n $번째 위치의 값을 계속 2등분해서 비교함으로써 분기가 가능하다는 것을 파악할 수 있다. 즉, 각 노드마다 모든 문자열 $ L $ 을 일일히 비교할 필요가 없다는 것이다.

물론 $ L = 4 $ 일 때, $ d = 4 $ 에서 $ d = 5 $ 로 넘어가기 위해서는 더 이상 $ n = 0 $ 번째의 문자의 분기가 불가능하기 때문에 다음 번째의 문자를 비교해야 하지만 상당히 빠른 것을 알 수 있다. 따라서 이를 일반화하면, 레드-블랙 트리에서 노드 깊이 $ d $ 와 문자열을 구성하는 집합의 크기 $ A $ 가 있을 때 분기를 위해서 필요한 문자 비교 횟수는 다음과 같다고 한다.
$$
\lceil \frac{d + 1}{log_{2}{|A|}} \rceil
$$

$$
\sum_{d = 0}^{log_2 n} \lceil\frac{d + 1}{log_2{|A|}}\rceil \approx 
\frac{(log_2{n})^2}{log_2{|A|}}
$$



그래서 레드-블랙 트리의 비교에 필요한 횟수는 길이 $ L $ 과 관계 없이, 항상 갯수 $ n $ 에 종속적임을 알 수 있다. 하지만 unordered_map 의 경우에는 $ L $ 에 직접적인 영향을 받기 때문에, 길이가 커질 수록 속도 저하가 커질수 있다.

하지만 이 경우는 모든 문자열이 골고루 나뉘어져 있을 때의 상황이고, 실제 상황은 약간 다른 경향을 보일 수 있다고 한다. 그렇지만 접두사가 비슷한 문자열들이 늘어져 있는 상황이 아니라면 대개 개수가 작은 상황에서는 map 이 unordered_map 보다는 성능이 더 좋을 수 있다고 한다.

### ! const char* 은 키로 쓰지 말 것.

* Stackoverflow 의 [글](https://stackoverflow.com/questions/41197296/does-c-string-hashing-hash-the-string-or-the-memory-address)

옛날 C 에서는 문자열을 나타내기 위해서는 char[] 배열형을 썼거나 const char* 을 썼었기 때문에 (const 자체는 C++ 에서 역수입한 것이지만) 차라리 키 타입으로 std::string 대신에 const char* 을 써볼까도 생각을 했었다. 하지만 이는 잘못된 것이고 성능을 좋게 한답시고 쓰는 것은 절대 아니된다고 한다.

왜냐면, const char* 는 문자열로도 쓸 수 있지만, 사실은 **문자열의 시작 위치를 가지는 Pointer** 이고, 정수형이나 별 다를바가 없기 때문이다. 만약에 다음과 같은 코드가 있다고 하면,

``` c++
std::unordered_map<const char*,int> map;
char str[11] = "bad";
map[str] = 2;           // hashes str = char*
auto x = map["bad"];    // hashes address of "bad"; x!=2
```

3번째 줄에서 "bad" 을 키로 넘길려고 할 때 (그리고 그렇게 될 거라고 생각할 때), 사실은 "bad\0" 이 넘겨지는 것이 아니라 'b' 을 가리키는 어떠한 포인터가 키로 넘겨져서 해쉬화 된다.

그래서 키를 넣고 룰루랄라하고 스트링 리터럴 "bad" 을 이용해서 저장한 값을 꺼낼려고 하면 참조하는 포인터가 다를 수 있기 때문에 런타임 에러가 터질 수도 있고, 디버깅을 하느라 밤을 새는 나날을 보내야 할 수도 있다. 물론 Visual C++ 의 경우에는 컴파일러 옵션으로 `/GF` 을 넣으면 동일한 포인터를 가리키게 할 수도 있지만 이럴 바에야 그냥 std::string 을 이용하는 게 낫다고 한다.

따라서 위의 실험에서 const char* 을 키로 받아서 쓰는 것이 std::string 보다 상당히 빨랐던 이유는 사실은 포인터의 값을 받아서 해쉬화 했기 때문이다.

## 결론

* 데이터 크기가 작을 때는 map, 클 때는 unordered_map
* map 은 레드-블랙 트리를 가져서 여러 노드를 계속 방문해야 하기 때문에 표본이 많을 수록 캐시 미스의 영향이 뚜렷하다.
* 문자열 키를 사용할 때는 map 이 더 많은 표본까지 우위를 가진다.
* const char* 은 실제로는 포인터이기 때문에 키로 썼다가는 큰일이 일어난다.