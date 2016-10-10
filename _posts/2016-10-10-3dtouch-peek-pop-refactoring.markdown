---
layout: post
title:  "3Dタッチ, Peek&Popのリファクタリング"
date:   2016-10-01 00:00:00 +0900
categories: swift
---

iPhone7も販売が開始して、3Dタッチを利用できる端末が増えてきました。  
3Dタッチの実装は、  
**previewingContext(_ previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController?**
このデリゲートメソッドの扱いが肝になるかなと思います。  
このメソッドで実行することとしては、下記３点あると思っています。

1. データオブジェクトの取得
2. previewContextのsourceRectの指定
3. ViewControllerの作成, データオブジェクトの引き渡し

そんな中、サンプルにあるようなtableViewの実装に関しては、  
indexPathForRow(at: location)でIndexPathを取り出し、  
データソースからindexPath.rowを用いてデータを取得する実装が多いかと思います。

{% highlight swift %}
extension ViewController: UIViewControllerPreviewingDelegate {
    func previewingContext(_ previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController? {
        guard let indexPath = self.collectionView.indexPathForItem(at: location) else { 
            return nil 
        }
        guard let cell = collectionView.cellForItem(at: indexPath) else { 
            return nil 
        }
        previewingContext.sourceRect = cell.frame
        let vc = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "PreviewViewController") as! PreviewViewController
        vc.photo = photos[indexPath.row]
        return vc
    }
    ・・・
}
{% endhighlight %}

これをProtocol + Protocol Extensionを利用してリファクタリングしてみます。

{% highlight swift %}

protocol Previewable {
    associatedtype T
    func fetchPreviewableContent(location: CGPoint) -> T?
    func fetchSourceRect(location: CGPoint) -> CGRect
    func previewingViewController() -> UIViewController?
}

{% endhighlight %}

**Previewable**というプロトコルを定義して、1,2,3を行う関数を定義してみます。

**fetchPreviewableContent**はCGPointからPreview ViewControllerに引き渡すデータソースを取得します。  
**fetchSourceRectはpreviewContext**のsourceRect指定に使用するCGRectを取得します。  
**previewingViewController**はプレビューするUIViewControllerを返すようにします。  

{% highlight swift %}

extension ViewController: Previewable {
    typealias T = Photo
    func fetchPreviewableContent(location: CGPoint) -> Photo? {
        guard let indexPath = self.collectionView.indexPathForItem(at: location) else {
            return nil
        }
        return self.photos[indexPath.row]
    }
    
    func fetchSourceRect(location: CGPoint) -> CGRect {
        guard let indexPath = self.collectionView.indexPathForItem(at: location) else {
            return CGRect.zero
        }
        guard let cell = self.collectionView.cellForItem(at: indexPath) else {
            return CGRect.zero
        }
        return cell.frame
    }

    func previewingViewController(object: Photo) -> UIViewController {
        let vc = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "PreviewViewController") as! PreviewViewController
        vc.photo = object
        return vc
    }
}

extension ViewController: UIViewControllerPreviewingDelegate {
    func previewingContext(_ previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController? {
        guard let photo = fetchPreviewableContent(location: location) else {
            return nil
        }
        let vc = previewingViewController(object: photo)
        return vc
    }
    ・・・
}
{% endhighlight %}

またデータオブジェクトが単一ではなく複数種類返し,オブジェクトに応じてPreviewするViewControllerを切り分ける必要がある場合は、下記のように実装します。

{% highlight swift %}

extension DetailViewController: Previewable {
    typealias T = Any
    
    func fetchPreviewableContent(location: CGPoint) -> Any? {
      // do something to retrieve an any type object.
    }

    func previewingViewController(object: Any) -> UIViewController? {
        if let photo = object as? Photo {
            let vc = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "PhotoPreviewViewController") as! PhotoPreviewViewController
            vc.photo = photo
            return vc
        } else if let user = object as? User {
            let vc = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "UserPreviewViewController") as! UserPreviewViewController
            vc.user = user
            return vc
        }
        return nil
    }
}

{% endhighlight %}


まとめ  

* previewingContext(_ previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint)は責務が大きいので、役割を分離してあげたほうが見通しが良くなるかも。  
* previewActionを考慮するとブロックの引き渡しもできる必要がありそう・・・ 

参考資料  

* [Getting Started with 3D Touch](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/Adopting3DTouchOniPhone/)



