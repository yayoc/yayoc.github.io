---
layout: post
title:  "swift2でコマンドラインツールを作ってみる"
date:   2016-01-15 15:09:53 +0900
categories: jekyll 
---

[こちらの記事](:http://techlife.cookpad.com/entry/2015/11/09/150248)を参考に  
swiftを使って[コマンドラインツール](:https://github.com/yayoc/thumbnail-kit)を作ってみました。

```
./main.swift ~/sample.png
```

@3x画像サイズのリソースを引数に渡すことで、iOS画像リソース用の各サムネイルが生成されます。

#swiftをコマンドラインで実行する際のtips

##引数を受け取る

*Process.arguments*でコマンドライン引数を受け取ることができます。

{% highlight swift %}
let filename: String = Process.arguments[1]   
print(filename)
{% endhighlight %}

とすると

{% highlight shell %}
$./main.swift ~/sample.png    
~/sample.png
{% endhighlight %}

という風になります。

##exitする

{% highlight swift %}
exit(EXIT_SUCCESS)
exit(EXIT_FAILURE)
{% endhighlight %}

で処理をexitすることができます。

#まとめ
* よりswiftを身近に使える+コマンドラインからOS XのAPI触れるの楽しと思います。
* 業務レベルで採用するときは開発環境などの検討事項は多そうな気がします。



ref
[Swift2で作るコマンドラインツール](:http://techlife.cookpad.com/entry/2015/11/09/150248)



