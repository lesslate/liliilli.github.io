---
layout: single
title: "imgui + OpenGL導入メモ"
tags: OpenGL ImGui DianYing C++ Programming
date: 2018-08-28 +0900
categories: Development
comments: true
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
`DianYing`というレンダラーアプリケーションの開発途中でGUIを実装したいと思った。だけどQtはあまりにも思いし、いろいろC++のGUI実装ライブラリーで独特でいいものがないかと思いきゃ見つけたのが`ImGui`である。

> [ImGui](https://github.com/ocornut/imgui)

# 動かすには

今のプロジェクトで`OpenGL`の側は一時的に`GLFW`というフレームワークを使っている。そして`ImGui`ではそのフレームワーク＋コンテクストの実装ファイルがもう用意されているので以下のファイルを直接プロジェクトに追加する。

* `imgui.h` `.cpp`
* `imgui_draw.cpp`
* `imgui_demo.cpp`
* `imgui_impl_glfw.h` `.cpp`
* `imgui_impl_opengl3.h` `.cpp`

`ImGUi`はビルドしたライブラリーファイルの形では提供されていないので直接プロジェクトで挟んでビルドしないといけない。筆者はここでリンクエラーが出まくってわけわからなくて苦労しました。

## 初期化

``` c++
// IMGUI DEMO
IMGUI_CHECKVERSION();
ImGui::CreateContext();
ImGuiIO& io = ImGui::GetIO(); (void)io;

ImGui_ImplGlfw_InitForOpenGL(gGlWindow, true);
ImGui_ImplOpenGL3_Init("#version 430");
```

ウィンドウ側の`Init`とレンダリングコンテキスト側の`Init`を呼び出して`ImGui`の初期化を行う。その前に`ImGui::CreateContext()`は必須

閉じる時にはちゃんとリソースの解除をしておく。

``` c++
ImGui_ImplOpenGL3_Shutdown();
ImGui_ImplGlfw_Shutdown();
```

## ウィンドウにいろいろ表示するには

`Imgui::Begin()`と`ImGui::End()`を用いてその中でステックに個別ウィンドウの構造の実装をする。

``` c++
ImGui::Begin("Hello, world!");
ImGui::Text("This is some useful text.");
ImGui::Checkbox("Demo Window", &gImguiShowDemoWindow);
ImGui::Checkbox("Another Window", &gImguiShowAnotherWindow);

ImGui::SliderFloat("Float", &f, 0.0f, 1.0f);
ImGui::ColorEdit3("Clear color", gColor.Data().data());

if (ImGui::Button("Button"))
{
  ++counter;
}
ImGui::SameLine();
ImGui::Text("Counter = %d", counter);

ImGui::Text("Application average %.3f ms/frame (%.1f FPS)", 
            1000.0f /ImGui::GetIO().Framerate, ImGui::GetIO().Framerate);
ImGui::End();
```

実装が終わったら`ImGui::Render()`でレンダリング準備完了を、そしてレンダリングコンテキストなりの`RenderDrawData`関数で実レンダーリングを行う。

# 結果

![img28_imgui]({{ "/assets/201808/img28_imgui.jpg" | absolute_url }})

ちゃんと表示される。

# 反省点

実は`OpenGL`も`GLFW`じゃなくてネイティブ(Windowsだったら`WIN32`など)で描画するつもりだった。しかしちゃんと描画されてなくて臨時手当てで`GLFW`を使っている。後で`WIN32`で描画するのならまた別の実装が必要かもしれない。

# 参考

> [imgui導入メモ(directx11)](http://nanka.hateblo.jp/entry/2017/03/21/003719)