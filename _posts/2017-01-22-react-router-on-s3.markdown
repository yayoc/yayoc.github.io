---
layout: post
title:  "React RouterをAmazon S3で利用する"
date:   2017-01-22 00:00:00 +0000
categories: js
---

[React-Router](https://github.com/ReactTraining/react-router/tree/master/docs)をルーティングに利用した、SPAを 
Amazon S3でホスティングした際にハマったので。

多くのReactアプリケーションのルーティングでReact-Routerが使われているかと思うのですが、  
Amazon S3にホスティングして、アクセスしてみると、残念ながら`404`エラーになってしまいます。  
Amazon S3がバケット上にファイルの存在を確認できないため、`404`になっているのでした。  

対応としては、3つのオプションがあるのかなと思っていて、  
(他アイデアあれば教えていただきたい)

1. [Nginx](https://nginx.org)などのProxyサーバを立てて、index.htmlへ遷移させる
2. React-Rotuer `hashHistory`を利用する
3. [Amazon CloudFront](https://aws.amazon.com/cloudfront/)を利用する

今回は簡潔にAWS上だけでセットアップしたい思いがあったので、  
[Cloud Front](https://aws.amazon.com/cloudfront/)を利用することにしました。

対応した内容は簡単で、  
Cloud Frontの[Custom Error Response](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/custom-error-pages.html)を作成して、  
HTTP Error Codeが`404`の場合は、`200` でindex.htmlにルーティングしてあげるというものです。


![Imgur](http://i.imgur.com/He3lE12.png){:width="800px"}


この設定で問題なく、React-Routerの設定したルーティングでアクセスすることが可能になりました。

ただ、こちらの対応だと`404`のレスポンスが全て`200`で返ってしまうので、  
そもそもReact-Routerを利用しないという方法を別途検討しています。


参考資料 

* [React router on Amazon Amazon S3](http://aserafin.pl/2016/03/23/react-router-on-amazon-Amazon S3/)


