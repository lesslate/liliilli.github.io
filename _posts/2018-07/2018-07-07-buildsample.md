---
layout: single
title: "LunarG/VulkanSample 윈도우에서 빌드하기"
tags: 
date: 2018-07-07 +0900
categories: Vulkan
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

`Vulkan Programming Guide` 라는 책을 기껏 샀더니만, 원 저작자가 아직도 소스를 올리지 않고 있어서 궁여지책으로 잘 알려진 샘플 코드 레포지터리인 [https://github.com/LunarG/VulkanSamples](https://github.com/LunarG/VulkanSamples) 나 [https://github.com/SaschaWillems/Vulkan](https://github.com/SaschaWillems/Vulkan) 을 알아보게 되었다. 그런데 `LunarG` 의 벌칸 샘플을 빌드할려고 하니 `MSBUild.exe`가 없다고 하면서 에러를 뿜는 것이다.

그래서 검색을 해본 결과, [https://stackoverflow.com/questions/46223916/msbuild-exe-not-found-cmd-exe](https://stackoverflow.com/questions/46223916/msbuild-exe-not-found-cmd-exe) 에서 답을 찾았는데, 일단 `vswhere` 로 비주얼 스튜디오가 설치된 장소를 알아내서 거기의 `bin` 폴더에 `MSBuild.exe` 을 환경변수로 추가하면 파워쉘과 같은 커맨드 창에서 `msbuild` 을 쓸 수 있다는 것을 알게 되었다.

![unity3]({{ "/assets/201807/img07_vkUpdate.PNG" | absolute_url }})

그렇게 해서 `bin` 폴더를 환경변수에 추가한 후, 다시 지시대로 샘플 레포지터리를 빌드를 하니 잘 돌아가는 것을 확인할 수 있었다.

+ 그런데... 이번엔 `MSBuild` 쪽에서 에러가 나는 것을 확인했다.

``` text
"D:\Development\TutorialProjects\VulkanSamples\build\ALL_BUILD.vcxproj"(기본 대상)(1)->
"D:\Development\TutorialProjects\VulkanSamples\build\submodules\Vulkan-LoaderAndValidationLayers\submodules\googletest\
googletest\gtest.vcxproj"(기본 대상)(74)->
(ClCompile 대상) ->
  d:\development\tutorialprojects\vulkansamples\submodules\vulkan-loaderandvalidationlayers\submodules\googletest\googl
etest\include\gtest\internal\gtest-internal.h : warning C4819: 현재 코드 페이지(949)에서 표시할 수 없는 문자가 파일에 들어 있습니다. 데이터가 손실되지 않게
하려면 해당 파일을 유니코드 형식으로 저장하십시오. [D:\Development\TutorialProjects\VulkanSamples\build\submodules\Vulkan-LoaderAndValidation
Layers\submodules\googletest\googletest\gtest.vcxproj]


"D:\Development\TutorialProjects\VulkanSamples\build\ALL_BUILD.vcxproj"(기본 대상)(1)->
"D:\Development\TutorialProjects\VulkanSamples\build\submodules\Vulkan-LoaderAndValidationLayers\submodules\googletest\
googletest\gtest.vcxproj"(기본 대상)(74)->
(ClCompile 대상) ->
  d:\development\tutorialprojects\vulkansamples\submodules\vulkan-loaderandvalidationlayers\submodules\googletest\googl
etest\include\gtest\internal\gtest-internal.h : error C2220: 경고가 오류로 처리되어 생성된 'object' 파일이 없습니다. [D:\Development\Tutori
alProjects\VulkanSamples\build\submodules\Vulkan-LoaderAndValidationLayers\submodules\googletest\googletest\gtest.vcxpr
oj]

    경고 1개
    오류 1개
```

이 오류는 쉘 인코딩이 `CP949` 등의, 기본 인코딩이 유니코드가 아닌 OS 혹은 쉘에서 빌드를 할 때 나타나는 에러로 보인다. `MSBuild` 에서는 이 `warning as errors` 을 억제할 방법은 없다. 그렇다고 해서 이 빌드 에러를 해결할 수 없는 것은 아니다.

우선 `vk_layer_validation_test` 와 `vk_loader_validation_tests`, 그리고 서브모듈 디렉터리에 있는 `gTest` 의 모든 프로젝트의 컴파일러 옵션에서 `/Wx-` 을 붙이자. 그리고 빌드를 다시 하면 다음과 같이 경고만 뜨고 에러가 튀어나오지는 않는다.

``` text
d:\development\tutorialprojects\vulkansamples\submodules\vulkan-loaderandvalidationlayers\submodules\googletest\googlet
est\include\gtest\internal\gtest-internal.h : warning C4819: 현재 코드 페이지(949)에서 표시할 수 없는 문자가 파일에 들어 있습니다. 데이터가 손실되지 않게 하려
면 해당 파일을 유니코드 형식으로 저장하십시오. [D:\Development\TutorialProjects\VulkanSamples\build\submodules\Vulkan-LoaderAndValidationLa
yers\tests\vk_layer_validation_tests.vcxproj]

...

    경고 10개
    오류 0개

경과 시간: 00:02:55.06
PS D:\Development\TutorialProjects\VulkanSamples\build>
````