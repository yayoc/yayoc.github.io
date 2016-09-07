---
layout: post
title:  "Swift3のアクセスコントロールについて"
date:   2016-09-07 10:00:00 +0900
categories: swift
---

XCode Beta 6がリリースされて、Swift3のアクセスコントロールまわりに変更があったので、まとめてみます。

## Swift2.2 におけるアクセスコントロール

* **Public**
  エンティテイはどのファイルからもアクセスすることができます。
* **Internal**
  エンティテイは同一モジュール内からからアクセスすることができます。
* **Private**
  定義している同一のファイルからのアクセスのみ許可します。

## Swift3.0 におけるアクセスコントロール

Swift2.2のprivate修飾子は**fileprivate**という修飾子になります。
> fileprivate: symbol visible within the current file

Swift2.2までのprivate修飾子は
> private: symbol visible within the current declaration

となり、例えばSwift2.2のときは、下記のように同一ファイル名であればアクセスできていたprivate変数も

{% highlight swift %}
class Person {
    private var name = "Jimi Hendrix"
}

extension Person {
    func introduce() {
        print("I'm \(self.name)")
    }
}
{% endhighlight %}

Swift3では、*'name' is inaccesible due to 'private' protection level*
というエラーを吐いてしまいます。

既存のコードに対する影響は大きく、Proposalにも下記のように書かれています。
> The existing code will need to rename private to fileprivate to achieve the same semantics. In many cases the new meaning of private is likely to compile as well and the code will then run exactly as before.

## Open修飾子

fileprivate以外にも既存のpublicアクセス修飾子の役割を分離するために  
**open**(classのメンバー変数をオーバライドできるアクセス修飾子)という  
新しいアクセスコントロールも追加されています。

>  It creates a new access level open beyond public; for now, open can only be used on classes and overridable class members.

## まとめ

Swift3のアクセスレベルは下記のようになるようです。

```
open > public > internal > fileprivate > private
```

参考資料
* [0117-non-public-subclassable-by-default.md](https://github.com/apple/swift-evolution/blob/master/proposals/0117-non-public-subclassable-by-default.md?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
* [0025-scoped-access-level.md](https://github.com/apple/swift-evolution/blob/master/proposals/0025-scoped-access-level.md?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
* [Swift 3 Access Control (Xcode 8 Beta 6)](https://swifting.io/blog/2016/08/17/22-swift-3-access-control-beta-6/?utm_campaign=This%2BWeek%2Bin%2BSwift&utm_medium=email&utm_source=This_Week_in_Swift_100)




