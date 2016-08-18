---
layout: post
title:  "Docker Containerを全削除・ストップする"
date:   2016-08-18 10:00:00 +0900
categories: docker
---

最近、Dockerを使って開発環境を作ることになり、メモ。

## 停止
{% highlight shell %}
docker stop $(docker ps -a -q)
{% endhighlight %}

## 削除
{% highlight shell %}
docker rm $(docker ps -a -q)
{% endhighlight %}

Docker, Vagrantより気軽に使えて便利な印象。
