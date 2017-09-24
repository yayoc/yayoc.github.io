---
layout: post
title:  "Webサイトのパフォーマンス向上。レンダリングの最適化について"
date:   2017-09-24 00:00:00 +0900
categories: performnace
---

先日、[tokyoto.js](https://kyotojs.connpass.com/event/64310/)でパフォーマンスについて ["What I did to improve performance score on Lighthouse"] (https://speakerdeck.com/yayoc/what-i-did-to-improve-performance-score-on-lighthouse) というタイトルで簡単にLTさせてもらったのですが、
レンダリング視点からのパフォーマンス向上について、この記事でもまとめてみたいと思います。

### How browser is rendering your site

レンダリングの向上について話す際に、いかにブラウザがサイトをレンダリングするかという仕組みを理解することが大事だと考えています。

#### DOM + CSSOM = Render tree

[Ilya Grigorik](https://twitter.com/igrigorik?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor)氏が書いた[Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)の記事が大変わかりやすいのですが、 
Browserはサーバからのhtmlレスポンスを受け取ったあと、
HTMLマークアップは**DOM (Document Object Model)**に変換され、CSSは**CSSOM(CSS Object Model)**を生成します。
( 各DOM, CSSOMへの変換の流れは[こちらの記事](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction)を参照することをおすすめします。 )

生成されたDOM, CSSOMはその後、render treeとして構成され、スクリーンに表示されることになります。
基本的には、CSSはrender blockとして扱われます。つまり、ブラウザはCSSを読み込みCSSOMを生成するまでレンダリングを開始しないということです。

また、JavaScriptに関しても同様にrender blockとなることがあります。
JavaScriptは動的言語として、DOMの変更であったり、スタイルの変更を容易に行え、パワフルなのですが、
ブラウザは**script tag**を見つけると、async属性などが明記されていない限り、JavaScriptのロード・実行が完了するまで
DOM構築を停止してしまいます。
また、仮にJavaScriptをCSSOMが構築される前に実行しようとすると、その実行はCSSOMが構築されるまで待機させられてしまいます。

**このように、DOM, CSSOM とJavaScriptの実行に関しては複雑な関係性があります。**

#### SPA, 複雑化するフロントエンド

一方、近年のフロントエンド界隈を取り巻くトレンドとして、SPAなどの
リッチなUIを実現してほしいという要求はますます強くなっています。
その影響もあり、JavaScript・CSSファイルサイズは肥大化しやすい傾向にありますし、パフォーマンスについて考えることも求められていると感じます。

では、レンダリングブロックを避けて、初回表示を高速化するためにどのようなことが考えられるでしょうか。
実際にLighthouseを計測ツールとして、Performanceスコアで80点以上を出すために行ったことが下記になります。

### What I did to optimize critical rendering path

#### 1. Removing CSS from rendering blocks

まず、スタイルの読み込みについてです。
CSSは先述の通り、render blockとして扱われますので、可能な限り小さいサイズにしてCSSOMをできるだけ早く構築することを目標とします。
ここでは、初期表示に必要な最低限のCSSと後から読み込みでも問題ないスタイルの切り分けを行い、
対応前は50KBあった一つのCSSファイルを分割することで、1KBほどまでサイズダウンしたCritical CSSをロードするだけでよくなりました。

#### 2. Reducing bundle size of JavaScript

JavaScriptに関しても、可能な限りファイルサイズを減らし、読み込みから実行までの時間を減らすことが求められます。
module bundlerとして**Webpack**を利用しているので、下記の機能を積極的に使うことで、JavaScriptの読み込み・実行を改善しました。

##### Tree shaking

**Tree shaking**はWebpack 2から提供されている機能で、ES2015モジュールとして提供されているファイルのimport/export構文を解析して、
アプリケーション内でdead codeになっていないかを判定します。
dead codeとして判定されたコード群は、 トランスパイル後、UglifyJSPluginなどを使い削除することができますし、全体のファイルサイズを小さくすることができます。

##### Code splitting

Code SplittingはDynamic import構文を利用することで、巨大なJavaScriptファイルをChunkという複数のファイル群に分割することができます。Code Splittingは共通のコードを共通Chunkとして取り出すためだけではなく、必要に応じた分割ができる機能を提供しています。つまり、例えば、リクエストURLベースに応じた必要最低限のコード群だけを提供することで、初回にロードするJavaScriptファイルのサイズを減らすことが可能となります。　

##### Scope hoisting

Webpack 3 で提供されたこの機能は`import`構文の解析により、コードベースの変更無しで単一のクロージャにコンバートすることを実現します。これまでのWebpackではモジュール一つ一つをクロージャとすることで、モジュールの独立性を保っていましたが、
Scope Hoistingを利用することで、実行速度の向上とファイルサイズの削除にも貢献します。

#### 3. Showing fake header at first

現在作成しているサイトではApplication Shellをデザインベースとして構成されており、
Header部分にはロゴとユーザのログイン情報などを表示するためのメニューボタン、
コンテンツ部分には実際のコンテンツを表示しています。

ここでのFake Headerというのは、疑似ヘッダーをCritical Rendering Pathに加えることで、ユーザ体験を向上させるものです。
Reactなど仮想DOMを利用したページレンダリングにおける、JavaScriptのロード完了までのblankページは、ユーザ体験として嬉しいものではありません。

疑似Headerをhtmlで構成し、JavaScript実行までは疑似ヘッダーを表示し、
JavaScript読み込みが完了後、動的情報も付与されたHeaderでオーバラップする工夫を行っています。体感としては、blankページの表示時間を減らすことができます。

#### 4. Using Intersection Observer 

Intersection Observerはオブザーブしている要素がスクリーン上に表示されているかどうかを簡単に抽出することができるAPIとなります。
こちらのAPIを利用することで初期表示時に不必要なコンテンツのロードを避けることができます。

### まとめ

レンダリング視点でのパフォーマンス向上に関して、render blockをどれだけ取り除けるか常に考える必要があると感じています。
まずは、不必要にsyncでJavaScriptを読んでいないか、JavaScript, CSSが肥大化していないかなどです。
Webpackなどが提供する機能は強力ですので、状況に合わせて対応することで簡単にパフォーマンスは向上できると思います。　

参考  

* [What I did to improve performance score on Lighthouse](https://speakerdeck.com/yayoc/what-i-did-to-improve-performance-score-on-lighthouse)



