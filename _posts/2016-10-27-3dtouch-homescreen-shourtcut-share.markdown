---
layout: post
title:  "3Dタッチ, ホームスクリーンのQuickActionについて"
date:   2016-10-25 00:00:00 +0900
categories: swift
---

アプリアイコンを3Dタッチすることで、QuickActionを表示することができます。
設定するには主に２つの方法があり、

1. StaticアクションをInfo.plistに指定する 
2. DynamicアクションをUIApplicationのshortcutItemsプロパティに指定する 

という方法があります。
ここで重要なのは

* Staticなアクションを指定する場合は、Dynamicなアクションよりも優先して表示される  
* QuickActionは最大4つまでしか登録できない(Shareのぞく)  
* Dynamicアクションは一度アプリを起動しないと表示されない  

ということです。

また、各アプリに表示される「Share :appname」のアクションに関しては、 
Developmentのプロビジョニングプロファイルで実機ビルドした場合は表示されず、  
Storeからダウンロードしたものについて付与されるようです。

実装は[サンプル](https://developer.apple.com/library/content/samplecode/ApplicationShortcuts/Introduction/Intro.html#//apple_ref/doc/uid/TP40016545)を参考にするといいかと思います。

参考資料  

* [Getting Started with 3D Touch](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/Adopting3DTouchOniPhone/)
* [3DTouch iOS Human Interface Guidelines](https://developer.apple.com/ios/human-interface-guidelines/interaction/3d-touch/)

