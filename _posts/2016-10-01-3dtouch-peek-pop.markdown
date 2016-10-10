---
layout: post
title:  "hitTest(_:with:)でPeek&Popを実現する"
date:   2016-10-01 00:00:00 +0900
categories: swift
---

iPhone7も販売が開始して、3Dタッチを利用できる端末が増えてきました。
そんな中、サンプルにもあるようなセルに対して1:1でデータを持つtableViewの実装に関しては、
** func previewingContext(_ previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController? **
のなかで、indexPathForRow(at: location)みたいにIndexPathを取り出し、
データソースから対象のデータを取り出して、Preview ViewControllwerに渡すようになるのかなと思っています。

{% highlight swift %}
    func previewingContext(_ previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController? {
        guard let indexPath = self.collectionView.indexPathForItem(at: location) else { return nil }
        guard let cell = collectionView.cellForItem(at: indexPath) else { return nil }
        previewingContext.sourceRect = cell.frame
        let vc = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "PreviewViewController") as! PreviewViewController
        vc.photo = photos[indexPath.row]
        return vc
    }
{% endhighlight %}

この方法で、ほとんどのアプリは問題なく実装できるだろうと思うのですが、例外もあって(実際にプロジェクトであった)
セルが複数の情報を持っている場合、locationに応じて、peekしているビュー情報を取得する必要があるケースがありました。

具体的には下記画像のようなセルを持つ場合、
ユーザアイコンをピークしている際はユーザ情報をプレビューしたい、アイテム情報をピークしている場合はアイテム情報をプレビューしたいといったケースです。




このときにホントはやりたくないですが、hitTest(_:with:)を使うことで解決できそうです。

まとめ
* hitTest(_:with:)でピークしているビューを抽出することで、複雑なセルに対するプレビュー機能を実装する
* セルに対して、プレビューケースが複数あるというUI自体がどうなんだろという問題もありそうな気もします・・・


参考資料  

* [hitTest(_:with:)](https://developer.apple.com/reference/uikit/uiview/1622469-hittest)




