---
layout: single
title: "ES - Block covering"
tags: Algorithm
date: 2018-10-12 +0900
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
#include <functional>
#include <vector>
#include <random>
#include <sstream>
#include <stack>
#include <string>

#define _MIN_    // Input parameter
#define _MOUT_   // Output parameter
#define _MINOUT_ // Input & Output parameter
 
#define M_NOTINITIALIZED -1
#define M_BEGINEND(__MAInstance__) __MAInstance__.begin(), __MAInstance__.end()
#define M_GET2D(__MAInstance__, __y__, __x__) __MAInstance__[__y__][__x__]
#define M_GET2DPOS(__MAInstance__, __MADPos__) \
  __MAInstance__[__MADPos__.mY][__MADPos__.mX]

/// Sample input string.
constexpr const std::string_view inputString = R"dy(3
3 7
#.....#
#.....#
##...##
3 7
#.....#
#.....#
##..###
8 10
##########
#........#
#........#
#........#
#........#
#........#
#........#
##########
)dy";

/// String stream.
std::stringstream sStringStream;
/// Forward declaration;
struct DMap;
/// Type aliasing

/// Global variables.
int32_t sCount = 0;

//!
//! Structures
//!

struct DPos
{
  int32_t mX = M_NOTINITIALIZED;
  int32_t mY = M_NOTINITIALIZED;
  DPos() = default;
  DPos(_MIN_ const int32_t x, _MIN_ const int32_t y) : mX{x}, mY{y} {};

  friend DPos operator+(const DPos& lhs, const DPos& rhs) noexcept
  {
    return DPos{lhs.mX + rhs.mX, lhs.mY + rhs.mY};
  }

  friend DPos operator-(const DPos& lhs, const DPos& rhs) noexcept
  {
    return DPos{lhs.mX - rhs.mX, lhs.mY - rhs.mY};
  }
};

struct DBlock
{
  std::array<DPos, 3> mCellPoss;
};

struct DMap
{
  enum EState { Wall, Empty, Block };

  int32_t mX = M_NOTINITIALIZED;
  int32_t mY = M_NOTINITIALIZED;
  std::vector<std::vector<EState>> mMap = {};
  int32_t mNumEmptyBlock = M_NOTINITIALIZED;

  DMap(_MIN_ int32_t x, _MIN_ int32_t y) : mX { x }, mY { y }
  {
    this->mMap.resize(this->mY);
    for (auto& row : this->mMap) { row.resize(this->mX, DMap::EState::Wall); };
    this->mNumEmptyBlock = 0;
  }

  void Push(_MIN_ const DBlock& block)
  { // Assertion & Body
    for (const auto& cellPos : block.mCellPoss)
    {
      assert(M_GET2DPOS(this->mMap, cellPos) == DMap::EState::Empty); 
      M_GET2DPOS(this->mMap, cellPos) = DMap::EState::Block;
      this->mNumEmptyBlock -= 1;
    }
    this->mBlockStack.push(block);
  }

  void Pop()
  { // Assertion
    assert(this->mBlockStack.empty() == false);
    // Body
    const auto& top = this->mBlockStack.top();
    for (const auto& cellPos : top.mCellPoss)
    {
      assert(M_GET2DPOS(this->mMap, cellPos) == DMap::EState::Block); 
      M_GET2DPOS(this->mMap, cellPos) = DMap::EState::Empty;
      this->mNumEmptyBlock += 1;
    }
    this->mBlockStack.pop();
  }

private:
  std::stack<DBlock> mBlockStack = {};
};

//!
//! Functions
//! 

DMap Setup()
{
  int32_t x, y;
  sStringStream >> y >> x;

  sCount = 0;
  DMap mapInstance{x, y};

  // Set map cell information
  for (int32_t i = 0; i < y; ++i)
  {
    std::string rowMap; sStringStream >> rowMap;
    for (int32_t r = 0; r < x; ++r) 
    { 
      if (rowMap[r] != '#')
      {
        M_GET2D(mapInstance.mMap, i, r) = DMap::EState::Empty;
        mapInstance.mNumEmptyBlock += 1;
      }
    }
  }

  return mapInstance;
}

static std::array<DBlock, 4> sDirectionBlock = {};
void __SetupDirectionBlocks()
{
  /// ##
  ///  #
  sDirectionBlock[0].mCellPoss[0] = DPos{0, 0};
  sDirectionBlock[0].mCellPoss[1] = DPos{1, 0};
  sDirectionBlock[0].mCellPoss[2] = DPos{1, 1};

  /// ##
  /// #
  sDirectionBlock[1].mCellPoss[0] = DPos{0, 0};
  sDirectionBlock[1].mCellPoss[1] = DPos{1, 0};
  sDirectionBlock[1].mCellPoss[2] = DPos{0, 1};

  /// #
  /// ##
  sDirectionBlock[2].mCellPoss[0] = DPos{0, 0};
  sDirectionBlock[2].mCellPoss[1] = DPos{0, 1};
  sDirectionBlock[2].mCellPoss[2] = DPos{1, 1};

  /// X#
  /// ##
  sDirectionBlock[3].mCellPoss[0] = DPos{0, 0};
  sDirectionBlock[3].mCellPoss[1] = DPos{0, 1};
  sDirectionBlock[3].mCellPoss[2] = DPos{-1, 1};
}

bool __GetEmptyCellPosition(_MIN_ const DMap& input, _MIN_ const DPos& cursor, 
                            _MOUT_ DPos& outputPos)
{ // Same row line
  for (int32_t x = cursor.mX; x < input.mX; ++x)
  {
    if (M_GET2D(input.mMap, cursor.mY, x) == DMap::EState::Empty) 
    {
      outputPos.mX = x;
      outputPos.mY = cursor.mY; 
      return true; 
    }
  }
  // Another row line and columns.
  for (int32_t y = cursor.mY + 1; y < input.mY; ++y)
  {
    for (int32_t x = 0; x < input.mX; ++x)
    {
      if (M_GET2D(input.mMap, y, x) == DMap::EState::Empty) 
      {
        outputPos.mX = x;
        outputPos.mY = y; 
        return true; 
      }
    }
  }

  return false;
}

bool __CheckAbletoInsert(_MIN_ const DMap& input, _MIN_ const DPos& cursor, 
                         _MIN_ const int32_t dir, _MOUT_ DBlock& block)
{ // Assertion
  assert(dir < 4);
  // Body
  for (int32_t i = 0; i < 3; ++i)
  {
    const auto refPos = cursor + sDirectionBlock[dir].mCellPoss[i];
    if (refPos.mX < 0 || refPos.mY < 0 || 
        refPos.mX >= input.mX || refPos.mY >= input.mY) { return false; }
    if (M_GET2DPOS(input.mMap, refPos) != DMap::EState::Empty) { return false; }
  }

  block.mCellPoss[0] = cursor + sDirectionBlock[dir].mCellPoss[0];
  block.mCellPoss[1] = cursor + sDirectionBlock[dir].mCellPoss[1];
  block.mCellPoss[2] = cursor + sDirectionBlock[dir].mCellPoss[2];
  return true;
}

void __Process(_MIN_ DMap& input, _MIN_ const DPos& cursor)
{
  for (int32_t dir = 0; dir < 4; ++dir)
  {
    DBlock block;
    if (__CheckAbletoInsert(input, cursor, dir, block) == false) { continue; }

    // If insertable, insert to stack and update map information.
    // and goto next stage...
    input.Push(block);
    
    DPos nextEmptyPosition = {};
    if (__GetEmptyCellPosition(input, cursor, nextEmptyPosition) == false) { 
      // After iterating, check that map has empty cell.
      // If not empty, just remove one step and update map information.
      if (input.mNumEmptyBlock == 0) { sCount += 1; }
      input.Pop();
      break; 
    }
    
    __Process(input, nextEmptyPosition);
    input.Pop();
  }
}

void BoardCover(_MIN_ DMap& input)
{ 
  DPos output;
  __GetEmptyCellPosition(input, DPos{0, 0,}, output);

  // Try first insert!
  __Process(input, output);
  std::printf("%d\n", sCount);
}

int main()
{
  __SetupDirectionBlocks();
  sStringStream << inputString.data();
  
  int32_t caseCount = M_NOTINITIALIZED; sStringStream >> caseCount;
  neu::test::Execute(&Setup, &BoardCover, "Test", caseCount);
}
```