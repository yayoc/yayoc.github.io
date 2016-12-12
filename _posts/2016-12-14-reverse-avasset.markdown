---
layout: post
title:  "AVAssetReader, AVAssetWriterを利用して逆再生動画を生成する"
date:   2016-12-14 00:00:00 +0000
categories: swift
---

![!Imgurl](http://i.imgur.com/B4V6jLp.gif){:width="400px"}

動画の逆再生処理を実装するために、[AVUtilities](https://github.com/whydna/Reverse-AVAsset-Efficient)というObjCで書かれたものがあったので、それをSwiftで置き換えてみました。

やっていることは大まかに下記になります。

2. AVAssetReaderを利用して、CMSampleBufferRefの配列を読み込み
3. 画像データを抜き出し、反転する
3. AVAssetWriterを使ってoutputURLのファイルに書き出し
4. 書き出しが完了したらcompletionHandlerを実行

実際にコードを見たほうが早いと思うので、こちらに。

{% gist fcbfe0de3f7a7b076647169cb45f58b3 %}

こんな感じで実行できます。

{% highlight swift %}
AVUtilities.reverse(originalAsset, outputURL: output, completion: { [unowned self] (reversedAsset: AVAsset) in
   // do something.
})
{% endhighlight %}


参考資料  

* [Reversing videos efficiently with AVFoundation](http://www.andyhin.com/post/5/reverse-video-avfoundation)
* [yayoc/Reverse-AVAsset-Efficient](https://github.com/yayoc/Reverse-AVAsset-Efficient) 
* [Reverse app](https://itunes.apple.com/jp/app/reverse-reverse-movie-camera/id1173143338?l=en&mt=8)
