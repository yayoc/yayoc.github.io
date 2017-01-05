---
layout: post
title:  "Reduxで利用する定数を楽に書く"
date:   2017-01-05 00:00:00 +0000
categories: log
---

[**Redux**](http://redux.js.org/) を使った実装をしていると、
`action.type`を下記のように定数で宣言し、

{% highlight js %}
export const ADD_TODO = 'ADD_TODO';
export const TOGGLE_TODO = 'TOGGLE_TODO';
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';
{% endhighlight %}

`reducers`, `actions`などでは以下のようにimportする実装がよく書かれていると思うのですが  
( Redux のドキュメントでもこのような実装だった )
{% highlight js %}
import { ADD_TODO, TOGGLE_TODO, SET_VISIBILITY_FILTER } from './actions'
{% endhighlight %}

宣言するaction.typeが増えた場合に、  
定数の宣言に不自由さを感じたので、  
定数を作るための関数を用意しました。

{% highlight js %}
export function createConstants(...constants) {
  return constants.reduce((accumulator, constant) => {
    accumulator[constant] = constant;
    return accumulator;
  }, {});
}
{% endhighlight %}

定数の宣言はこの関数を使うと下記のようになります。

{% highlight js %}
import { createConstants } from '../utils';

export default createConstants(
  'ADD_TODO',
  'TOGGLE_TODO',
  'SET_VISIBILITY_FILTER',
);

{% endhighlight %}

各`reducers`, `actions`では下記のようにimportすることで、シンプルに定数を扱うことができます。

{% highlight js %}
import C from '../constans';

switch (action.type) {
  case C.ADD_TODO:
  ...
}
{% endhighlight %}


ちなみに、`createConstants`関数は[eslint-confing-airbnb](https://www.npmjs.com/package/eslint-config-airbnb)にあてると、
`no-param-reassign`で

```
Assignment to property of function parameter 'accumulator'
```

とerrorになるので、`.eslintrc`に下記を追加して、errorを避けています。

{% highlight json %}
"rules": {
   "no-param-reassign": [2, {
     "props": false
   }]
 }
{% endhighlight %}
