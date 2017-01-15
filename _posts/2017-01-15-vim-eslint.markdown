---
layout: post
title:  "VimとESLintを連携させる"
date:   2017-01-15 00:00:00 +0000
categories: vim
---

テキストエディタにVimを使っています。 
JavaScriptのlinterには[ESLint](http://eslint.org/)を利用しており、  
VimとESLintの連携を行うとコーディングが捗ったので、その設定を。 

## Pluginの導入

ESLintとVimを連携させるには下記のプラグインをインストールします。 
僕の場合はプラグインマネージャーに[Vundle](https://github.com/VundleVim/Vundle.vim)を利用しているので、  
下記のプラグインを`.vimrc`に追記して、`:PluginInstall`を実行します。 

{% highlight vim %}
Plugin 'vim-syntastic/syntastic.git'
{% endhighlight %}

## Syntasticの設定

ESLintのチェック結果をプレビューするために下記の設定を行います。

{% highlight vim %}

" ESLint configuration
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*

let g:syntastic_always_populate_loc_list = 1
let g:syntastic_loc_list_height = 5
let g:syntastic_auto_loc_list = 0
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 1
let g:syntastic_javascript_checkers = ['eslint']

let g:syntastic_error_symbol = '❌'
let g:syntastic_style_error_symbol = '⁉️'
let g:syntastic_warning_symbol = '⚠️'
let g:syntastic_style_warning_symbol = '💩'

highlight link SyntasticErrorSign SignColumn
highlight link SyntasticWarningSign SignColumn
highlight link SyntasticStyleErrorSign SignColumn
highlight link SyntasticStyleWarningSign SignColumn

{% endhighlight %}

## eslint-cliの導入

ESLintがglobalにインストールされている場合は、ローカルのESLintを参照するように
[eslint-cli](https://www.npmjs.com/package/eslint-cli)を追加します。

```
npm install -g eslint-cli
```

## Vim

これで、ESLintがファイル保存時に対象のJavaScriptファイルをチェックするようになります。 
`:Error`コマンドでエラー内容を表示することもできます。

![Imgur](http://i.imgur.com/MwK0FUg.png){:width="800px"}

### エラー結果が表示されない場合

eslintが動いていない可能性があるので、対象ファイルに対して直接下記コマンドラインで実行してみることをおすすめします。

```
eslint somefile.js
```


参考資料  

* [In Editor Linting with Syntastic](http://usevim.com/2016/03/07/linting/)


