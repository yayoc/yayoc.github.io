---
layout: post
title:  "Fetch API response.json()のレスポンスがPromise objectを返す"
date:   2016-12-09 00:00:00 +0900
categories: JavaScript
---

JavaScriptで提供されているFetch APIを使う事があったので、メモ。

## Fetch API

> The Fetch API provides an interface for fetching resources (including across the network). It will seem familiar to anyone who has used XMLHttpRequest, but the new API provides a more powerful and flexible feature set.

これまでHttpリクエストで使っていた、XMLHttpRequestの代わりとなる、新しいAPIです。
一部ブラウザではまだサポートされていませんが、このへんは[Polyfill](https://en.wikipedia.org/wiki/Polyfill)を使って解消できます。

今回は簡単なGETのAPIリクエストを実装したのですが、
その中でResponseの挙動が独特だなと思ったので、メモしています。

idからユーザ情報を取得する実装を行う場合下記のような実装が考えられます。
(reduxの実装でのサンプルになります。)

{% highlight javascript %}
export function getUser(id) {
    return dispatch =>
      fetch(`http://localhost:3000/users/${id}`, {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
        }
      })
      .then(response => {
        if (response.status >= 200 && response.status < 300) {
          response.json().then(res => {
            dispatch(doSomethingSuccessfully(res));
          });
        } else {
          const error = new Error(response.statusText);
          error.response = response;
          dispatch(doSomethingFailed(error));
          throw error;
        }
      })
      .catch( error => {
          console.log(error);
      });
}
{% endhighlight %}

この中で、`response.json()`まわりの実装が独特だなと思っていて、
この場合の`response.json()`は`Promise object`を返すように設計されています。

この設計の意図は、どうやら**responseのjsonがInvalidの場合は、Rejectとして扱ったほうがいいということで、Promise objectを返しているようです。**

.json()関数は、responseボディを受け取るためのPromiseを得るために実行する関数となります。
また、fetch API でのresponseはhttpヘッダーを受け取ったときに扱うことができます。
Promise objectで返すことによって、Chainableなネストしないコードが書けます。

というわけで、Fetch APIでresponse bodyを扱う場合は、下記のように実装するとよいかと思います。

{% highlight javascript %}
fetch(url).then(response =>
  response.json().then(data => ({
    data: data,
    status: response.status
  })
).then(res => {
  console.log(res);
});
{% endhighlight %}

参考  

* [Fetch API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API)  
* [Why does .json() return a promise if in an object literal?](https://getpocket.com/a/read/1503781158)
