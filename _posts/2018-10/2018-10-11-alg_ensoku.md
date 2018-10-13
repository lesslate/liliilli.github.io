---
layout: single
title: "Ensoku Exhausitive Search"
tags: Algorithm
date: 2018-10-11 +0900
categories: Algorithm
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

``` c++
/// Sample input string.
constexpr const std::string_view inputString = R"dy(3
2 1
0 1
4 6
0 1 1 2 2 3 3 0 0 2 1 3
6 10
0 1 0 2 1 2 1 3 1 4 2 3 2 4 3 4 3 5 4 5
)dy";

struct DStudent;
using TContainer = std::vector<DStudent>;
using TIndex = int32_t;

/// Student information
struct DStudent final
{
  std::vector<TIndex> mFriendIdList = {};
};

/// String stream.
std::stringstream sStringStream;
int32_t sCount;

void __AddFriendRelationShip(_MIN_ TContainer &input, 
                             _MIN_ const TIndex lhs, _MIN_ const TIndex rhs)
{
  if (lhs < rhs) { input[lhs].mFriendIdList.emplace_back(rhs); }
  else 					 { input[rhs].mFriendIdList.emplace_back(lhs); }
}

TContainer Setup()
{
  int32_t friendCount, pairCount;
  sStringStream >> friendCount >> pairCount;

  sCount = 0;
	TContainer studentList;
	studentList.resize(friendCount);

  for (int32_t i = 0; i < pairCount; ++i)
  {
    int32_t lhs, rhs;
		sStringStream >> lhs >> rhs;
		__AddFriendRelationShip(studentList, lhs, rhs);
	}

	for (auto &student : studentList) { std::sort(M_BEGINEND(student.mFriendIdList)); }
	return studentList;
}

bool __IsAlreadyBinded(_MIN_ const std::vector<TIndex> &bindedList, 
											 _MIN_ const TIndex val)
{
	return std::find(M_BEGINEND(bindedList), val) != bindedList.end();
}

void __Process(_MIN_ TContainer &input, 
							 _MIN_ const int32_t cursorId, _MIN_ const int32_t remainGoal, 
							 _MOUT_ std::vector<TIndex> &bindedList)
{
	if (remainGoal == 0) { sCount += 1; return; }

	const int32_t size = static_cast<int32_t>(input.size());
	for (TIndex i = cursorId; i < size; ++i)
	{
		// Chk:: If toward student id is already binded to list,
		if (__IsAlreadyBinded(bindedList, i) == true) { continue; }

		// (1) Input
		bindedList.emplace_back(i);

		// (2) Iterate
		const int32_t toSize = static_cast<int32_t>(input[i].mFriendIdList.size());
		for (TIndex to = 0; to < toSize; ++to)
		{
			// Chk:: If toward friend id is already binded to list,
			const auto rhsId = input[i].mFriendIdList[to];
			if (__IsAlreadyBinded(bindedList, rhsId) == true) { continue; }

			// (3) Insert binded list
			bindedList.emplace_back(rhsId);
			__Process(input, i + 1, remainGoal - 1, bindedList);
			bindedList.pop_back();
		}

		// (5) Pop back.
		bindedList.pop_back();
	}
	return;
}

void Picnic(_MIN_ TContainer &input)
{
	const int32_t pairGoal = static_cast<int32_t>(input.size()) / 2;
	std::vector<TIndex> bindedList = {};

	__Process(input, 0, pairGoal, bindedList);
	std::printf("%d\n", sCount);
}

int main()
{
	sStringStream << inputString.data();

	int32_t caseCount = M_NOTINITIALIZED;
	sStringStream >> caseCount;
	neu::test::Execute(&Setup, &Picnic, "Test", caseCount);
}
```