---
layout: post
title:  "React Router Linkコンポーネントを動的に表示する"
date:   2017-06-11 00:00:00 +0900
categories: js
---

SNSのコメント情報などに含まれるハッシュタグ(#javaScriptとか)・ユーザ名(@yayocとか)を動的にリンクに変換したい場面で、リンクを[React Router](https://github.com/ReactTraining/react-router)の[Link](http://knowbody.github.io/react-router-docs/api/Link.html)コンポーネントで表したい場合の実装。

```js
  linkedComment = (comment: string) => {
    const re = /([^@#]+)|([@#]{1}[^@#()\s]+)|([#+])|([@+])/g;
    const found = comment.match(re);
    if (found === null) return <span>{comment}</span>;

    const nodes = found.map(v => {
      if (v.search(/[@#]/i) === -1) return <span>{v}</span>;
      if (v.length <= 1) return <span>{v}</span>;

      const link = v.substring(0, 1) === "#"
        ? `/tag/${v.substr(1, v.length)}`
        : `/n/${v.substr(1, v.length)}`;
      return <Link to={link}>{v}</Link>;
    });
    return nodes;
  };
```

与えられた文字列に対して、該当するフォーマット`/([@#]{1}[^@#()\s]+)/`であれば、Linkコンポーネントを生成。当てはまらなければ、spanタグを返すという実装になっています。

ただの正規表現です。ハッシュタグ, @マークが並んだ際の無効化は考慮しています。
(正規表現もっとうまくかける気がしますが・・・)
![Imgur](http://i.imgur.com/SxqPcOy.png)

本件とは関係ないですが、
ブログ更新滞っていたので、積極的に更新していきます💪
