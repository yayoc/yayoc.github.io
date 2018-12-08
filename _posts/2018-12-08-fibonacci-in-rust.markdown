---
layout: post
title:  "fibonacciをRustで書いてみる"
date:   2018-12-08 00:00:00 +0900
categories: rust
---

最近、Rustを勉強しています。[Rust Book](https://doc.rust-lang.org/book/)をあらかた読み終わった後、  
[O'REILLY](https://www.safaribooksonline.com/home/)でRustのチュートリアル動画などを読んだり、
[Let's build a browser engine](https://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html)でRustで書かれたコードを読んで[TS](https://github.com/yayoc/crusoe)に書き直したりしています。

今回は簡単にRustでFibonacciを書いてみようと思います。  
素直に書くと下記のようになり、計算量はO(2^n)です。

```rust
const FIB_ZERO: u64 = 1;
const FIB_ONE: u64 = 1;

fn fib(n: u64) -> u64 {
    if n == 0 {
        FIB_ZERO
    } else if n == 1 {
        FIB_ONE
    } else {
        fib(n - 1) + fib(n - 2)
    }
}

fn main() {
    for i in 0..50 {
        println!("{} {}", i, fib(i));
    }
}
```

これを、HashMapを利用して、メモ化するように書き直してみます。  
こちらの実装では、計算量をO(n)にすることができます。

```rust
use std::collections::HashMap;

fn fib_with_hash(n: u64, map: &mut HashMap<u64, u64>) -> u64 {
    match n {
        0 | 1 => 1,
        n => {
            if map.contains_key(&n) {
                *map.get(&n).unwrap()
            } else {
                let val = fib(n - 1, map) + fib(n - 2, map);
                map.insert(n, val);
                val
            }
        }
    }
}
```

[criterion](https://crates.io/crates/criterion)を使ってベンチーマックを取ってみます。


```sh
fib                     time:   [3.9633 ms 3.9882 ms 4.0144 ms]
                        change: [+32.973% +34.683% +36.571%] (p = 0.00 < 0.05)
```

```sh
fib with hash           time:   [61.156 us 61.767 us 62.405 us]
                        change: [+6.5985% +11.279% +15.608%] (p = 0.00 < 0.05)
```

Timeでみると61.1us VS 3.9msなので63倍近い差が出ました。  
Rust関係ないブログになってしまいましたが、引き続きRustを使っていきたいなと。

参考

* [ アルゴリズム百選 - フィボナッチ数列にO()を学ぶ ](http://blog.livedoor.jp/dankogai/archives/50958771.html)
* [Rust book](https://doc.rust-lang.org/book/)
* [Let's build a browser engine](https://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html)
* [Learning Rust](https://www.safaribooksonline.com/videos/learning-rust/9781788477918)

