---
layout: single
title: "Clang 6.0.1とVS2017との連携の仕方"
tags: Clang MSVC
date: 2018-08-27 +0900
categories: C++
comments: true
---
<script 
  type="text/javascript"
  src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

VisualC++コンパイラーはMSVCで基本付きのC++プログラミング言語のコンパイラーで、今最新バージョンではC++17までの全ての機能を支援しています。（ClangやGccではまだできてないアルゴリズムの並列ポリシーなども支援しています。）

ですけど問題はVisualC++はWindowsでしか使えませんのでLinuxまたはUnix系列のOSではClangやgccを使わないといけません。そしてVC++（VisualC++）は自分なりの雑な機能もついてあり、Visual Studioで正常に動作したプロジェクトが他のコンパイラーなどではエラーを吐き出す場合もありえます。それに加えてコンパイル速度も遅いです。

それであれこれして実際にWindows環境でVisual Studio 2017を使いながらClangを使おうと思いました。ですがVisual Studio 2017で支援しているLLVMフレームワークはバージョンが4.0.1で止まっています。普通に使うなら4.0.1でもいいんですけど出来れば最新のバージョンを使いたいですね。それでv6.0.1をMSVCに適用しようと思っています。

# 手順

1. [LLVMサイト](http://releases.llvm.org/download.html)でLLVM v6.0.1をダウンロードします。いちいちコンパイルすることは面倒くさいしPre-Built BinariesでWindows (64-bit)をクリックしてインストールします。
2. インストールしてからMSBuildツール及びLLVMが環境変数にちゃんとバインディングされているか確認します。
3. VS2017で提供するLLVMツールセットをインストールする代わりに[https://github.com/arves100/llvm-vs2017-integration](https://github.com/arves100/llvm-vs2017-integration)に入ってREADME通りにインストールします。（自分はオートインストールに失敗して手動でしました。）
4. 全てのインストールが終わったらC++のプロジェクトを開いてプロジェクトの`Property`窓に入ります。ここで一般（General）で`Platform Toolset`を自分の環境に合うLLVMツールセットに変更します。
5. `Property > C/C++ > Language`にて`Conformance mode`を`No`にします。ClangはまだC++17までの全ての標準を全部支援しきれませんので、`Yes(-permissive)`にするとエラーが出てしまいます。
6. ビルドすると問題なくビルドできます。サブシステムを`Windows`に変えてエントリーポイントを`WinMain()`にして`WIN32API`使いまくっても構いませんでした。

お疲れ様でした。