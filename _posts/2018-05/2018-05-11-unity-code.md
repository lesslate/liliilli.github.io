---
layout: single
title: Unity script editor 을 VS code 로 바꾸기
tags: Unity
categories: Unity
comments: false
date: 2018-05-11 13:23:23
---
　
<!-- more -->

> ***참고***
>
> Unity Development with VS Code
> https://code.visualstudio.com/docs/other/unity
>
> 유니티에 비주얼 스튜디오 코드 (Visual Studio Code; VSCode) 연동방법
>  https://ijemin.com/blog/%EC%9C%A0%EB%8B%88%ED%8B%B0%EC%97%90-%EB%B9%84%EC%A3%BC%EC%96%BC-%EC%8A%A4%ED%8A%9C%EB%94%94%EC%98%A4-%EC%BD%94%EB%93%9C-visual-studio-code-vscode-%EC%97%B0%EB%8F%99%EB%B0%A9%EB%B2%95/

Unity 를 설치하면 기본으로 바인딩되는 에디터가 Visual Studio 나 XCode 일 것이다. 그런데 유니티 프리팹 혹은 *GameObject* 에 바인딩된 스크립트를 고칠 때마다 무겁디 무거운 IDE 을 켜야한다면 시간 낭비이고 매번 고통이다. 그래서 코드 에디터인 비주얼 스튜디오 코드를 사용해서 에디터 설정을 바꿔보았다.

## 과정

![Externel Script Editor](..\..\..\..\..\images\201805\11\1525696548769.png)
**Externel Script Editor** 의 드롭다운 항목을 열어서, Browse 로 Visual Studio Code 의 실행 파일을 찾는다.

![File Explorer](..\..\..\..\..\images\201805\11\1525696713869.png)

*Code.exe* 을 찾아서 바인딩한다.

![Binded!](..\..\..\..\..\images\201805\11\1525696776838.png)

외부 스크립트 에디터로 바인딩이 된 것을 볼 수 있다.