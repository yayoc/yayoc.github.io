---
layout: post
title:  "UITableViewCellをカスタムカラーでハイライトする方法と注意点"
date:   2016-07-11 09:26:53 +0900
categories: swift
---

UITableViewCellの**selectionStyle**を使うとSDKで用意されている下記のカラーに変更することが可能です。

{% highlight swift %}

public enum UITableViewCellSelectionStyle : Int {
    
    case None
    case Blue
    case Gray
    @available(iOS 7.0, *)
    case Default
}

{% endhighlight %}

ただし、上記方法だと用意されたカラー以外の指定はできないので、上記以外のカラーを利用したい場合には
**selectedBackgroundView**を利用すると便利です。  
UITableViewDataSource ProtocolのcellForRowAtIndexPath内等でcellに対して 
**selectedBackgroundView**を設定します。

{% highlight swift %}
func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCellWithIdentifier("SampleTableViewCell", forIndexPath: indexPath) as? SampleTableViewCell
    let selectedBackgroundView: UIView = UIView()
    selectedBackgroundView.backgroundColor = UIColor.redColor()
    cell?.selectedBackgroundView = selectedBackgroundView
    return cell!
}
{% endhighlight %}

こうすることで、ハイライト時の背景色を指定することが出来ます。

![Imgur](http://i.imgur.com/e37NmAE.gif?1){:width="300px"}

しかしながら、UITableViewCellはハイライト・選択時にsubviewsの背景色をclear colorにしてしまうので、
この実装方法だけだと、UITableViewCellにサブビューがある場合、サブビューの背景色も置き換わってしまいます。  

![](http://i.imgur.com/ct1EFp8.png){:width="400px"}

例えば、上記のようなsubviewを置いた状態で挙動を確認するとサブビューの背景色がclear colorになってしまうので、背景がすべて赤色になったように見えます。 

![Imgur](http://i.imgur.com/aPJ4dtI.gif){:width="300px"}

これを避けるために、UITableViewCellの**setHighlighted**メソッドをオーバライドして下記のように実装します

{% highlight swift %}
override func setHighlighted(highlighted: Bool, animated: Bool) {
    super.setHighlighted(highlighted, animated: animated)
    self.subView.backgroundColor = UIColor.lightGrayColor()
}
{% endhighlight %}

![Imgur](http://i.imgur.com/Q9AYef0.gif){:width="300px"}

subviewの背景色が書き換わらないようになりました。

