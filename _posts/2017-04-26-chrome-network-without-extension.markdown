---
layout: post
title:  "ChromeのNetworkパネルからExtensionのリクエストを隠す方法"
date:   2017-04-26 00:00:00 +0900
categories: chrome
---

メモ。  
フロントエンドのデバッグ時などによく利用する`Network`タブですが、  
Chrome Extension のリクエストが表示されて、可読性が低いのがつらいポイントです。

![Imgur](http://i.imgur.com/tz898Ks.png)

ExtensionからのXHRリクエストをフィルタリングするために、  
`Filter`に`-scheme:chrome-extension`を入力すると、Extensionからのリクエストを非表示にすることができます。  

![Imgur](http://i.imgur.com/46W98GV.png)

しかしながら、このフィルタリングをデフォルトに設定する方法、また[API](https://developer.chrome.com/extensions/devtools_panels)がないため、 
インスペクタの起動時に毎回入力する必要がありそうです。


参考  

* [Add a way to hide/filter out extension resources in the network tab of web inspector	](https://bugs.chromium.org/p/chromium/issues/detail?id=239401)
* [How to set the default filter of network in Chrome Developer Tools?](http://stackoverflow.com/questions/37592092/how-to-set-the-default-filter-of-network-in-chrome-developer-tools)



