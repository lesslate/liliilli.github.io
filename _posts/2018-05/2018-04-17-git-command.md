---
layout: single
title: よく使うのに忘れてるgitのコマンドのメモ。
tags: git
categories: git
comments: false
date: 2018-04-17 14:49:40
---


## 始め

個人プロジェクトにGitを活用しながらお役に立ちそうなコマンドをいろいろ調べてまとめてみた。
<!--more-->

## 本文

{% highlight git %}
git branch -av
{% endhighlight %}

リモートを含めた全てのブランチのより詳しく把握するときに使われる。

{% highlight git %}
git log -S <検索するキーワード>
{% endhighlight %}

コミットコメントにキーワードがあるコミットだけを選別して見せる。

{% highlight git %}
git diff --staged [<他のオプション>...]
{% endhighlight %}

現在のStageにAddしたファイルの変更履歴などが見れる。
個人的には`--minimal`まで付けて入力するともっときれいに変更履歴が見れた。

{% highlight git %}
git add -i
{% endhighlight %}

より詳しくファイルをAddすることができる。それを用いてuntrackedの状況のファイルだけを追加することが簡単にできる。
他にもいろんな細部コマンドがあるようだが使ったことはあんまりない…。

{% highlight git %}
git push -d <レポジトリ> <削除したい遠隔ブランチ>
{% endhighlight %}

[git push](https://git-scm.com/docs/git-push)。中身は詳しく調べてなかったが`-d`オプションを付けて、そしてレポジトリと遠隔ブランチのネームを入力すると該当される遠隔ブランチが削除するようにアップデートするようだ。便利便利。

{% highlight git %}
git log --merges
{% endhighlight %}

マージコミットだけを見せる。逆に`-no-merges`を付けるとマージじゃない純粋なコミットだけを探してログとして出力する。

{% highlight git %}
git rebase -i HEAD~N
/*! and */
pick
squash
squash...
{% endhighlight %}

interactive rebaseを使うときによくお世話になるコマンド。HEADからN個前までのブランチをリベースする。`git squash`でも呼ばれるらしい？

## 参考

* [git squash - 여러개의 커밋로그를 하나로 묶기](http://meetup.toast.com/posts/39)
* [내 시간을 절약하는 git 명령어들](sunphiz.me/wp/archives/2558)