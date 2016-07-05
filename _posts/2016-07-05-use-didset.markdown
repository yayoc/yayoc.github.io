---
layout: post
title:  "didSetを使ってOutlet初期化の見通しを良くする"
date:   2016-07-05 11:26:53 +0900
categories: swift
---

Outletのレイアウトを初期化する際にviewDidLoadやinit処理で行うことが多いかもしれませんが、  
Swiftの場合ストアドプロパティ(lazy stored property以外)には  
[**didSet**/**willSet**](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-ID254)のProperty observersが用意されているので  
こちらを使って、Oultetのスタイルを初期化することも可能です。  

例えば、下記のような実装をしたCustomViewがあったとして、

{% highlight swift %}

import UIKit

class CustomView: UIView {
    @IBOutlet weak var backgroundView: UIView! 
    @IBOutlet weak var label: UILabel! 
    @IBOutlet weak var button: UIButton! 

    override init(frame: CGRect) {
        super.init(frame: frame)
 	configureLayouts()
    }
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
 	configureLayouts()
    }

    private configureLayouts() {
        backgroundView.backgroundColor = UIColor.blackColor()
        label.textColor = UIColor.blackColor()
        button.selected = true
    }	
}

{% endhighlight %}

didSetにレイアウトの初期化を追加すると下記のように変更できます。

{% highlight swift %}

import UIKit

class CustomView: UIView {
    @IBOutlet weak var backgroundView: UIView! {
        didSet {
            backgroundView.backgroundColor = UIColor.blackColor()
        }
    }
    @IBOutlet weak var label: UILabel! {
        didSet {
            label.textColor = UIColor.blackColor()
        }
    }
    @IBOutlet weak var button: UIButton! {
        didSet {
            button.selected = true
        }
    }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
}

{% endhighlight %}

didSetを利用すると各Outlet単位で初期化処理の見通しがよくなるかと思います。

ref: [iOS: A Beautiful Way of Styling IBOutlets in Swift](https://www.natashatherobot.com/ios-a-beautiful-way-of-styling-iboutlets-in-swift/)

