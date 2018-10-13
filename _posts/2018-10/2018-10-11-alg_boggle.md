---
layout: single
title: "Boggle (Brute-force)"
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

#include <algorithm>
#include <array>
#include <chrono>
#include <vector>
#include <random>
#include <functional>
#include <string>
#include <sstream>

#define _MIN_		 // Input parameter
#define _MOUT_	 // Output parameter
#define _MINOUT_ // Input & Output parameter

#define M_PRINT
#define M_NOTINITIALIZED -1

/* INPUT STRING WILL BE LIKE THIS.
1
URLPM
XPRET
GIAET
XTNZY
XOQRS
6
PRETTY
GIRL
REPEAT
KARA
PANDORA
GIAZAPX
*/

struct DPos
{
	int32_t x = 0;
	int32_t y = 0;

	friend DPos operator+(const DPos &lhs, const DPos &rhs) noexcept
	{
		return DPos{lhs.x + rhs.x, lhs.y + rhs.y};
	}

	friend DPos operator-(const DPos &lhs, const DPos &rhs) noexcept
	{
		return DPos{lhs.x - rhs.x, lhs.y - rhs.y};
	}
};

struct DChk
{
	std::string string = {};
	bool isExist = false;
};

std::array<DPos, 8> sDirections =
{
	DPos{-1, -1},DPos{0, -1},DPos{1, -1},
	DPos{-1, 0},             DPos{1, 0},
	DPos{-1, 1}, DPos{0, 1}, DPos{1, 1}
};

constexpr const std::string_view inputString = R"dy(1
URLPM
XPRET
GIAET
XTNZY
XOQRS
6
PRETTY
GIRL
REPEAT
KARA
PANDORA
GIAZAPX
)dy";

int32_t sCaseCount = M_NOTINITIALIZED;
std::vector<DChk> sCheckList = {};

using TContainer = std::vector<std::vector<char>>;
#define M_GETVALUE(__MAContainer__, __MAPos__) \
	__MAContainer__[__MAPos__.y][__MAPos__.x]

TContainer Setup()
{
	std::stringstream stringStream;
	stringStream << inputString.data();

	// Test case
	stringStream >> sCaseCount;
	std::printf("%d\n", sCaseCount);

	// Every bingo grid is 5x5.
	TContainer container;
	for (int32_t i = 0; i < 5; ++i)
	{
		std::string row;
		stringStream >> row;
		container.emplace_back();

		for (int32_t j = 0; j < 5; ++j)
		{
			container[i].emplace_back(row[j]);
		}
	}

	// Get check string list size.
	int32_t checkListSize = M_NOTINITIALIZED;
	stringStream >> checkListSize;

	// Get check strings
	for (int32_t i = 0; i < checkListSize; ++i)
	{
		std::string string;
		stringStream >> string;
		sCheckList.emplace_back(DChk{string, false});
	}

	return container;
}

[[nodiscard]] 
std::vector<DPos> __FindStartPoint(_MIN_ const TContainer &input, _MIN_ const char chr) 
{
	std::vector<DPos> result = {};

	for (int32_t y = 0; y < 5; ++y)
	{
		for (int32_t x = 0; x < 5; ++x)
		{
			if (input[y][x] == chr)
			{
				result.emplace_back(DPos{x, y});
			}
		}
	}

	return result;
}

[[nodiscard]] 
std::vector<DPos> __GetForwardingPosition(_MIN_ const DPos &currentPos)
{
	std::vector<DPos> resultPositions = {};

	for (int32_t i = 0; i < 8; ++i)
	{
		const DPos nextPos = currentPos + sDirections[i];
		// Out of bound check
		if (nextPos.x < 0 || nextPos.x >= 5)
		{
			continue;
		}
		if (nextPos.y < 0 || nextPos.y >= 5)
		{
			continue;
		}

		resultPositions.push_back(nextPos);
	}

	return resultPositions;
}

[[nodiscard]] 
bool __FindChar(_MIN_ const TContainer &input, _MIN_ const std::string &string,
								_MIN_ const int32_t remainCount, _MIN_ const DPos &currentPos) 
{
	if (remainCount == 0) { return true; }
	else
	{
		// First, find and get list of forwarding positions from currentPosition.
		const auto nextPositions = __GetForwardingPosition(currentPos);
		const int32_t nextCharId = static_cast<int32_t>(string.size() - remainCount);
		const char nextCharToFnd = string[nextCharId];

		for (int32_t i = 0; i < 8; ++i)
		{
			if (M_GETVALUE(input, nextPositions[i]) == nextCharToFnd)
			{
				return __FindChar(input, string, remainCount - 1, nextPositions[i]);
			}
		}

		return false;
	}
}

void Boggle(_MIN_ TContainer &input)
{
	const int32_t checkSize = static_cast<int32_t>(sCheckList.size());
	for (int32_t i = 0; i < checkSize; ++i)
	{
		// First, find start point"S". It is possible that multiple start points are exist.
		const auto &string = sCheckList[i].string;
		const char startChar = string[0];
		const auto startPoss = __FindStartPoint(input, startChar);
		const int32_t size = static_cast<int32_t>(startPoss.size());
		// If not found any start point, just leave as false value.
		if (size == 0) { continue; }
		// If start point is exist and string is just one character, change flag to true.
		if (string.size() == 1)
		{
			sCheckList[i].isExist = true;
			continue;
		}

		// General base case, find char using exhaustive search.
		for (int32_t j = 0; j < size; ++j)
		{
			const bool result = __FindChar(input, string, string.size() - 1, startPoss[j]);
			if (result == true)
			{
				sCheckList[i].isExist = true;
				break;
			}
		}
	}
}

int main()
{
	neu::test::Execute(&Setup, &Boggle, "Test", 1);

	for (size_t i = 0; i < sCheckList.size(); ++i)
	{
    std::printf("%s %s\n", sCheckList[i].string.c_str(), 
								sCheckList[i].isExist ? "YES" "NO");
	}
}
```