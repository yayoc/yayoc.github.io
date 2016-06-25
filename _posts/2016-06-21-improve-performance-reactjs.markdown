---
layout: post
title:  "React jsのパフォーマンス改善でやったこと"
date:   2016-06-26 05:26:53 +0900
categories: reactjs
---

Reactjs を利用したプロジェクトでパフォーマンス改善を行ったので、対応内容をまとめてみます。

主に行ったことは

- Reactjsのベンチマーク測定
- Shouldupdatecomponentの実装
- Immutable jsの採用

になります。

まず、reactjsのベンチマークを図るためにアドオンのperfを使います。
今回のプロジェクトではrailsで動かしていた都合
react-railsのgemを使っていたので、
development.rbに下記の1行を追加します。

{% highlight ruby %}
MyApp::Application.configure do
  config.react.addons = true
end
{% endhighlight %}

これで
[https://facebook.github.io/react/docs/addons.html](https://facebook.github.io/react/docs/addons.html)
こちらのaddonが使える様になります。

setStateして、レンダリングが完了するまでを測定するために下記のような処理を親コンポーネントに加えます。

{% highlight js %}
React.addons.Perf.start();
this.setState({ something: something }, function() {
  React.addons.Perf.stop();
  React.addons.Perf.printInclusive();
  React.addons.Perf.printWasted();
});
{% endhighlight %}

すると、chromeのコンソールログに下記のようなテーブルフォーマットで解析データを吐き出してくれます。

**Perf.printInclusive()**
はレンダリングに要した時間を全てひっくるめて表示します

**Perf.printWasted()**
は各コンポーネントがレンダリングされず、ユーザがDOM操作できない時間を表示します。

ベンチマークを元にボトルネックを探し出します。
ReactJSは**Shouldupdatecomponent**というライフサイクルイベントを初めから用意してくれているのですが、
この関数はコンポーネントが新しいprops, stateを受け取ったあとに、レンダリング前に呼ばれます。
そして、この関数でfalseを返すと、対象コンポーネントをレンダリングしないように制御することができます。
デフォルトでこの関数はtrueを返すようになっているので、この関数を実装しない場合は、対象コンポーネントは新たなprops, stateを受け取るたびに再レンダリングされてしまします。

そういったことから、ReactJSのパフォーマンスが気になった場合（特に、たくさんのコンポーネントを持っているとき）はshouldComponentUpdateを利用して、改善を図ることが可能です。

```
If performance is a bottleneck, especially with dozens or hundreds of components, useshouldComponentUpdate to speed up your app.
```

たとえば、下記のようなコンポーネントがあって、Add New Bookボタンを押すと、
 新規で追加するBookコンポーネント以外も再処理が走ります。

{% highlight js %}

var Books = React.createClass({
  getInitialState: function( ) {
    var book1 = { id: 1, title: 'book title1', price: 500, author: { name: 'yayoc1', gender: 'male' } };
    var book2 = { id: 2, title: 'book title2', price: 300, author: { name: 'yayoc2', gender: 'female' } };
    var book3 = { id: 3, title: 'book title3', price: 250, author: { name: 'yayoc3', gender: 'female' } };
    var books = [ book1, book2, book3 ];
    return {
      books: books
    };
  },
  addBook: function() {
    var lastBook = this.state.books[this.state.books.length - 1];
    var newBook = { id: lastBook.id + 1, title: `book title${lastBook.id+1}`, price: 1000, author: { name: 'yayoc_new', gender: 'female' } };
    var newBooks = this.state.books.concat([newBook]);
    React.addons.Perf.start();
    this.setState( { books: newBooks }, function() {
      React.addons.Perf.stop();
      React.addons.Perf.printInclusive();
      React.addons.Perf.printWasted();
    });
  },
  render: function() {
    return (
        <div>
          <button onClick={this.addBook}> Add new Book</button>
          <BookList books={this.state.books} />
        </div>
      )
  }
});

var BookList = React.createClass({
  render: function() {
    var bookNodes = this.props.books.map(function(book) {
      return <Book key={book.id} book={book} />
    });
    return (
        <ul>
            {bookNodes}
        </ul>
    )
  }
});

var Book = React.createClass({
  render: function() {
    return  (
      <li>
        <span>{this.props.book.title}</span>
        <span>{this.props.book.price}</span>
        <BookAuthor author={this.props.book.author} />
      </li>
    )
  }
});

var BookAuthor = React.createClass({
  render: function() {
    return (
      <div>
        <span>{this.props.author.name}</span>
        <span>{this.props.author.gender}</span>
      </div>
    )
  }
});
{% endhighlight %}

では、Bookの中に、
下記のようにShouldupdatecomponentを実装してみます。

{% highlight js %}

var Book = React.createClass({
  shouldComponentUpdate: function(nextProps, nextState) {
    return nextProps.book !== this.props.book;
  },
  render: function() {
    return  (
      <li>
        <span>{this.props.book.title}</span>
        <span>{this.props.book.price}</span>
        <BookAuthor author={this.props.book.author} />
      </li>
    )
  }
});

{% endhighlight %}

対象のコンポーネントだけが更新されるようになりました。

ただし、先ほど追加した下記の処理には問題があります。
たとえば、Bookのauthorオブジェクトだけを更新した際にも下記の処理はfalseを返してしまい、
コンポーネントが再描画されません。
また、それらの変更に対応するようにshouldComponentUpdateの処理を修正すると
バグを生みやすいコードが作られる可能性が高まってしまいます。

{% highlight js %}

shouldComponentUpdate: function(nextProps, nextState) {
    return nextProps.book !== this.props.book;
}

{% endhighlight %}

そこで、最後に、3のImmutableJSを採用する話になります。

ImmutableJSはFacebookが開発したライブラリで、List, Stack, Map, OrderMap, Set, OrderedSet, Recordといったイミュータブルなデータ構造を提供してくれます。Javascriptにおける、データのコピーや受け渡しで煩雑になる部分を最小化してくれるライブラリです。

shouldComponentUpdateの比較も下記のように簡単に変更することが可能になります。

{% highlight js %}

shouldComponentUpdate: function(nextProps, nextState) {
    return !Immutable.is(nextProps.book, this.props.book);
}

{% endhighlight %}

では、Immutable JSを利用するように上記のコンポーネントを書き換えます。

{% highlight js %}

var Books = React.createClass({
  getInitialState: function( ) {
    var book1 = { id: 1, title: 'book title1', price: 500, author: { name: 'yayoc1', gender: 'male' } };
    var book2 = { id: 2, title: 'book title2', price: 300, author: { name: 'yayoc2', gender: 'female' } };
    var book3 = { id: 3, title: 'book title3', price: 250, author: { name: 'yayoc3', gender: 'female' } };
    var books = [ book1, book2, book3 ];
    return {
      books: Immutable.fromJS(books)
    };
  },
  addBook: function() {
    var lastBookId = this.state.books.getIn([this.state.books.size - 1, "id"]);
    var newBook = Immutable.fromJS({ id: lastBookId + 1, title: `book title${lastBookId+1}`, price: 1000, author: { name: 'yayoc_new', gender: 'female' } });
    var newBooks = this.state.books.push(newBook);
    React.addons.Perf.start();
    this.setState( { books: newBooks }, function() {
      React.addons.Perf.stop();
      React.addons.Perf.printInclusive();
      React.addons.Perf.printWasted();
    });
  },
  render: function() {
    return (
        <div>
          <button onClick={this.addBook} > Add new Book</button>
          <BookList books={this.state.books} />
        </div>
      )
  }
});

var BookList = React.createClass({
  render: function() {
    var bookNodes = this.props.books.map(function(book) {
      return <Book key={book.get('id')} book={book} />
    });
    return (
        <ul>
            {bookNodes}
        </ul>
    )
  }
});

var Book = React.createClass({
  shouldComponentUpdate: function(nextProps, nextState) {
    return !Immutable.is(nextProps.book, this.props.book);
  },
  render: function() {
    return  (
      <li>
        <span>{this.props.book.get('title')}</span>
        <span>{this.props.book.get('price')}</span>
        <BookAuthor author={this.props.book.get('author')} />
      </li>
    )
  }
});

var BookAuthor = React.createClass({
  render: function() {
    return (
      <div>
        <span>{this.props.author.get('name')}</span>
        <span>{this.props.author.get('gender')}</span>
      </div>
    )
  }
});

{% endhighlight %}

まず、state booksで管理するデータをイミュータブルな状態にします。

**books: Immutable.fromJS(books)**

Immutableな状態にしたデータはgetメソッドを利用して取得します。

**this.props.book.get(‘title’)**

ネストしたデータ構造の場合はgetInを利用することで簡単に取り出せます。

**this.props.book.getIn([‘author’, ‘name’])**

これでデータをイミュータブルに扱えて、安心してshouldComponentUpdateを使えるようになりました。

まとめ

- ReactJSを利用してパフォーマンスが気になった場合は、shouldComponentUpdateを利用する
- データがイミュータブルでない場合、shouldComponentUpdateはバグを生みやすい
- ImmutableJSを採用するとそのあたりは気にしないで済むので楽
- ただし、ImmutableJSを後から導入すると移行が大変

以上になります
