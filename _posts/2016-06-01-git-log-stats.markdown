---
layout: post
title:  "git logでどれくらい仕事したか確認する"
date:   2016-06-01 16:09:53 +0900
categories: git log stats
---

git logどれくらい仕事したか確認したいときってあると思う(?)のですが、
コードのライン数で測るワンライナーです。

下記のコマンドを計測したいレポジトリで実行してあげると、
追加行と削除業を確認できます。   
**-- since** オプションで期間も変更可能です。

{% highlight bash %}
git log --numstat --pretty="%H" --author="AUTHOR NAME" --since 1.day | awk 'NF==3 {plus+=$1; minus+=$2} END {printf("+%d, -%d\n", plus, minus)}'
{% endhighlight %}

僕が作業しているとあるレポジトリで1週間のログを出すとこんな感じになりました。

{% highlight bash %}
$ git log --numstat --pretty="%H" --author="yayoc" --since 1.week | awk 'NF==3 {plus+=$1; minus+=$2} END {printf("+%d, -%d\n", plus, minus)}'
+2574, -1715
{% endhighlight %}







