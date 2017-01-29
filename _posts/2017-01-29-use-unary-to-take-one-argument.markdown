---
layout: post
title:  "Unaryデコレータを使って、map(parseInt)を期待通りに返す"
date:   2017-01-29 00:00:00 +0900
categories: JavaScript
---

今回はmap関数を利用した際にしばしば起きる問題についての対応を。  

![Imgur](http://i.imgur.com/kqdCcHr.png)

上記のように、  

{% highlight js %}
['1', '2', '3'].map(parseFloat)
{% endhighlight %}

は`[1, 2, 3]`という配列を返しているのに対して、

{% highlight js %}
['1', '2', '3'].map(parseInt)
{% endhighlight %}

は`[1, NaN, Nan]`という配列を返してしまいます。

これは、`Array.prototype.map()`は下記のような引数を取り、  

* callback  
	* currentValue  
	The current element being processed in the array.  
	* index  
	The index of the current element being processed in the array.  
	* array  
	The array map was called upon.  
* thisArg
	Optional. Value to use as this when executing callback.

また、[parseInt](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/parseInt)が第二引数に`radix`をとる関数なので、  
今回の場合、mapのコールバックで渡されるindexを`radix`として、parseIntに渡しているために  
paraseIntは数値に変換することができずNaNを返したわけです。


## Unary

今回はUnary関数を作成して、`['1', '2', '3'].map(parseInt)`の返り値を`[1, 2, 3]`にするようにします。  
`Unary`関数は関数の引数をひとつだけ取るように変更するためのデコレーターです。

{% highlight js %}

const unary = (fn) => 
  fn.length === 1 
    ? fn
    : function (something) {
      return fn.call(this, something);
    }

{% endhighlight %}

![Imgur](http://i.imgur.com/7v81u25.png)

この関数を使うことで期待通りの配列が返ってきました。  
直感的にかけるので、この書き方を気に入っています。

参考  

* [parseInt](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/parseInt)  
* [parseFloat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseFloat)
