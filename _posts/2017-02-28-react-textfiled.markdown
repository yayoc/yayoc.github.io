---
layout: post
title:  "react-textfieldというReact Componentを作ってみた"
date:   2017-02-28 00:00:00 +0900
categories: JavaScript
---

1ヶ月ぐらい前の話なんだけど、  
フロントエンドエンジニアとして、Reactを触るようになってからある程度ナレッジがたまったので、 
[react-textfield](https://www.npmjs.com/package/react-textfield) というReact Componentのnpm packageを作ってみた。

`Elegant text field`とか書いているけど、やっていることは単純で、validatorを内包したtextfieldである。  
一応、ちょっとしたvalidator群もAPIとして用意しているのだけど、足りない場合はPRを送っていただけると嬉しい。  
実際の利用サンプルなどは[こちら](https://github.com/yayoc/react-textfield/blob/master/examples/FormWrapper.js)にあげている  

ユーザのサインアップフォームとかログインフォームなどで設置して、簡単にsuccess/errorメッセージを表示できるようになっている。  

![image](https://raw.githubusercontent.com/yayoc/react-textfield/master/react-textfield-demo.gif)

今回のコンポーネント作成にあたり、[storybook](https://getstorybook.io/docs/react-storybook/basics/introduction)を使っての[デモページ](http://blog.yayoc.com/react-textfield/)作成を行ってみた。  
UIの実装をインタラクティブにできる + state確認, デモページのフォーマットが整えられるので、便利だなという印象。  

開発環境のセットアップに関しては、[Airbnb](https://airbnb.com)が出している[react-dates](https://github.com/airbnb/react-dates)を大いに参考にさせていだいた。

## まとめ

* `storybook`は手軽に使えて、デモページの作成には最適
* textfieldの単一コンポーネントを作ったが、Form単位で書き出したライブラリでないとなかなか使いにくいかもなので、時間があれば、作ってみたい
* フォーム周り、サイトのセンスが現れるので、丁寧に作り込んでいきたい


参考  

* [react-textfield](https://www.npmjs.com/package/react-textfield)
* [storybook](https://getstorybook.io/docs/react-storybook/basics/introduction)
* [react-dates](https://github.com/airbnb/react-dates)
