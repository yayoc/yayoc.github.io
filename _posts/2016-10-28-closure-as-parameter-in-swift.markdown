---
layout: post
title:  "Swiftの関数は必ず何らかの引数を取るということ"
date:   2016-10-28 00:00:00 +0900
categories: swift
---

Swiftを触っていて、割と初歩的なことを認識できていなかったので備考録。

## error: ambiguous use

下記のようなクロージャーを引数にする関数を持っているstructを実行すると

{% highlight swift %}

struct Music {
    static func play(completion: @escaping () -> Void) {
        print("🎵")
    }
}

Music.play(completion: { (something) in } )

{% endhighlight %}

結果は

{% highlight swift %}
🎵
{% endhighlight %}

になります。ここまでは期待通りの動作です。  

次に、この**Music** structに 
{% highlight swift %}
static func play(completion: @escaping (String) -> Void) {
    print("🎵🎵🎵")
}
{% endhighlight %}

というString型を引数にするクロージャを引数に指定した同名のメソッドを宣言してみると、下記のようなエラーメッセージが表示されます。 

{% highlight swift %}
Playground execution failed: error: MyPlayground.playground:10:1: error: ambiguous use of 'play(completion:)'
Music.play(completion: { (something) in } )
{% endhighlight %}

**() -> Void**に対して、引数を持たないクロージャという認識だったため、上記のエラーで混乱してしまいました。  
( そもそも引数を持たないという理解だったら、*Music.play(completion: { (something) in } )*で動くのがおかしいと思うべきだったかも・・)

## Swiftの関数は必ず何らかの引数を取る。

クロージャで指定した**() -> Void** について考えてみます。  
Swiftのモジュールを見ると

{% highlight swift %}
public typealias Void = ()
{% endhighlight %}

**Void**というのは**()**のエイリアスとなっています。  
また、**()型**というのは空のタプルを表しているので、**() -> Void** というのは空のタプルを受け取り、空のタプルを返すという事になります。  
[ドキュメント](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html)にも下記のように書かれています。 

> Functions without a defined return type return a special value of type Void. This is simply an empty tuple, which is written as ().

今回の場合、somethingという型の指定がない引数を取っていたため、このsomethingは**()型**も**String型**も取れる可能性があり、 
コンパイラがメソッドを指定できなかったという事になります。  
例えば、上記の処理を下記のように、型指定してあげるとエラーは修正されます。

{% highlight swift %}
Music.play(completion: { (r: String) in } )
Music.play(completion: { (r: ()) in } )
{% endhighlight %}

## まとめ

* Swiftの関数は必ず、何らかの引数を取り、何らかの返り値を返す。  
* () -> Void が引数を取らないクロージャという理解は良くない。 

参考資料  

* [Functions](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html)  
* [関数における引数/戻り値とタプルの関係](http://dev.classmethod.jp/smartphone/swift-function-tuple/)
