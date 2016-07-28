---
layout: post
title:  "UINavigationController の画面遷移でアニメーションを利用する"
date:   2016-07-21 17:26:53 +0900
categories: UIKit, iOS
---

UINavigationController のクラスメソッドであるpushViewController, popViewControllerを利用する際にカスタムアニメーションを
加える際に、下記のような実装を見かけました。
(アニメーションはUIViewController **presentViewController:animated:completion:** のモーダル表示をイメージしています。)

{% highlight swift %}

let transition = CATransition()
transition.duration = 0.3
transition.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)
transition.type = kCATransitionReveal
transition.subtype = kCATransitionFromBottom
self.navigationController?.view.layer.addAnimation(transition, forKey: nil)
self.navigationController?.pushViewController(toViewController, animated: true)

{% endhighlight %}

これでもアニメーションは実現できるかとは思うのですが、  
UINavigationControllerには **navigationController:animationControllerForOperation:fromViewController:toViewController**  
というdelegate methodが用意されているのでこちらを利用したほうがいいかなと思います。

{% gist 3a64b848ad9fbd4aa53f426e93fa7cdb %}




