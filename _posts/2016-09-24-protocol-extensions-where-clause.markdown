---
layout: post
title:  "Protocol Extensionsのwhere句を使ってTraitを実現し、BaseViewControllerを排除する"
date:   2016-09-24 00:00:00 +0900
categories: swift
---

iOSアプリの開発において共通処理をBaseViewController(仮)のような親クラスに定義して、 
各画面はこの親クラスを継承するといった実装がわりとあるかなと思います。  
このような実装の場合、BaseViewControllerが肥大化して、見通しの悪い設計になりがちです。  
また、独自フレームワークのような実装になってしまう傾向があり、  
チーム開発においては、メンバーの学習コストが上がるなどのデメリットがあります。

その一方、各viewControllerに共通で入れたい処理というのはあるので、  
そういった場合は、Protocol Extensionsの**Where句**を利用してTraitを実現するといいなと考えています。

例えば、**UIActivityViewController**を使って、データを共有する処理を共通化する場合    
BaseViewControllerを利用した実装は下記のようになるかなと思います。

{% highlight swift %}
import UIKit

class BaseViewController: UIViewController {
    func share(url: String) {
        if let url = URL(string: url) {
            let items = [url]
            let vc = UIActivityViewController(activityItems: items, applicationActivities: nil)
            self.present(vc, animated: true, completion: nil)
        }
    }
}

class ViewController: BaseViewController {
    func shareButtonPressed() {
        let url = "http://yayoc.com"
        share(url: url)
    }
}
{% endhighlight %}

これに対して、**Shareable**というprotocolを作り、  
UIViewControllerをwhere条件にextensionを追加するとこのように実装できます。

{% highlight swift %}
import UIKit

protocol Sharable {
    func share(url: String)
}

extension Shareable where Self: UIViewController {
    func share(url: String) {
        if let url = URL(string: url) {
            let items = [url]
            let vc = UIActivityViewController(activityItems: items, applicationActivities: nil)
            self.present(vc, animated: true, completion: nil)
        }
    }
}

class ViewController: UIViewController, Shareable {
    func shareButtonPressed() {
        let url = "http://yayoc.com"
        share(url: url)
    }
}
{% endhighlight %}

where句にUIViewControllerを指定することで、**present**メソッドが呼べるようになり、
各画面ではShareableプロトコルを定義すればshareメソッドを利用することができます。

このように、BaseViewControllerのような親クラスを利用せず、各画面への共通処理を実装したい場合は
protocol extension + Where句でTraitを実現すると依存度の低い実装が実現できそうです。

参考資料  

* [Mixins and Traits in Swift 2.0](http://matthijshollemans.com/2015/07/22/mixins-and-traits-in-swift-2/) < かなり良記事でした。
* [iOSアプリの設計でBaseViewControllerのようなのは作りたくない](http://qiita.com/yimajo/items/8ef1850e1362bd3bf3d8)
* [Swiftのprotocol extensionでmixin的なものを実現する](http://qiita.com/sagaraya/items/5e73501ff7a3f0c502c0#_reference-5d8e21ff15ca3b1c9023)




