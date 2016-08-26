---
layout: post
title:  "Objective-Cで書いたinstancetypeを返すメソッドがswiftから実行出来ない場合の対処"
date:   2016-08-18 10:00:00 +0900
categories: swift
---

SwiftとObjective-Cが共存するプロジェクトでAPI呼び出し部分で下記のような問題に直面しました。  

Objective-Cで実装されている下記のようなinstancetype型を返すクラスメソッドを    
Swiftから呼び出しを行おうとすると、コンパイラエラーが発生しました。

Objective-C interfface

{% highlight objc %}

@interface MyBook: NSObject
+ (instancetype)book;
@end

{% endhighlight %}

Swift 呼び出し

{% highlight swift %}

let mybook = MyBook.book()

{% endhighlight %}

```
'book()' is unavailable: use object construction 'MyBook()'
```

## なぜエラーになるか？

bookメソッドが下記条件を満たしているため、
コンパイラには*convinience method*として認識されてしまい、コンパイルエラーを吐いてしまっているようです。	

1. 引数を取っていない関数である
2. クラスの派生的な命名となっている

たとえば、下記のようなクラス名でinterfaceを持つ場合、

{% highlight objc %}

@interface LegacySomething: NSObject
+ (instancetype)something;
+ (instancetype)thing;
@end

{% endhighlight %}

Swiftで実行すると下記の結果となります。

{% highlight swift %}

let mySomething = LegacySomething.something() ## Error
let myThing = LegacySomething.thing() ## Success

{% endhighlight %}


## 解決するには

上記サンプルにもあるように命名を変更するだけで対応することが可能です。  
たとえば、UIApplicationで利用されているような*sharedApplication*のような命名は有効で、
コンパイラはこれがconvinienceメソッドだと認識しなくなるようです。

{% highlight objc %}

@interface UIApplication: NSObject
+ (instancetype)sharedApplication;
@end

{% endhighlight %}

{% highlight swift %}

let application = UIApplication.sharedApplication()

{% endhighlight %}

今回はinterfaceを拡張して無事Swiftから実行できるようになりました🎉

{% highlight objc %}

@interface MyBook: NSObject
+ (instancetype)book;
+ (instancetype)sharedBook;
@end

{% endhighlight %}


ref  
* [Swift's Objective-C bridging and initializers](http://justinmiller.io/posts/2015/01/28/swift-objc-initializers/)
* [iOS Dev Nugget 116 method() is unavailable: use object construction Class()](http://hboon.com/iosdevnuggets/ios-dev-nugget-116-method()-is-unavailable--use-object-construction-class()/)