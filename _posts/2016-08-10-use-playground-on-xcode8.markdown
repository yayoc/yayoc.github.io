---
layout: post
title:  "XCode8でPlaygroundを使ってTableViewを表示してみる"
date:   2016-08-10 10:00:00 +0900
categories: iOS
---

![Imgur](http://i.imgur.com/udD0xlx.gif){:width="400px"}

Playgroundを利用して簡易的にUIの確認が行えるのはいいなと思い、試してみました。
環境はXCode8.4 (Beta)です。

やっていることはほぼ、[こちらの記事](http://himaratsu.hatenablog.com/entry/playground)で書かれている内容です。
今回はXCode8で動かしているため、PlaygroundSupport moduleを利用しています。

## PlaygroundSupportについて

```
The PlaygroundSupport module provides behavior to help you take advantage of playgrounds. Use this module to:
```

PlaygroundSupportはXcode7で使われていたXCPlayground moduleの後継です。
主に、Playgroundを利用する上でのLive Viewなどを実現してくれるモジュールとなります。
XCPlaygroundに関してはXCode7でDeprecatedになるので、  
XCode8からはPlaygroundSupportを利用することになります。

## Live Viewを試す

Battles(バンド)が好きなので、今回はBattlesのアルバムタイトル一覧をUITableViewに表示してみます。

{% gist 3caad346787cd89883283ca0141cd2ab %}

![Imgur](http://i.imgur.com/G3s7ljh.gif){:width="400px"}

UITableViewCellのstyleを書き換えたり、cellのbackground colorを書き換えるとビルドせずにすぐさまlive Viewに反映されます。
UIのちょっとした確認が気軽にできるのは重宝しそうです。ただ、Xib, Storyboard使えないのでコードでレイアウトを表す必要があります。　
また、XCode8からはliveViewの指定が下記のようになるようです。

{% highlight swift %}
PlaygroundPage.current.needsIndefiniteExecution = true
PlaygroundPage.current.liveView = navigationController.view
{% endhighlight %}

## まとめ
* かなり気軽にUIを試せるのは重宝しそう。
* **Swift**の文法とか試すのにも良さそう。
* tableViewの実装は適当に書いています・・

ref  
[UIの確認をPlaygroundを使って行う](http://himaratsu.hatenablog.com/entry/playground)
[Battles](https://battles.bleepstores.com/)

