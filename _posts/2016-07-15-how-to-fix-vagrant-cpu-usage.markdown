---
layout: post
title:  "VagrantのCPU使用率が100%になったときに対応した内容"
date:   2016-07-15 09:26:53 +0900
categories: vagrant
---

[**Vagrant**](https://www.vagrantup.com/)はローカルの開発環境を作る際によく利用するのですが、  
CPUの利用率が100%になる問題に直面したので、対応した内容をまとめておきます。

1. NFSを利用する
2. システムメモリの1/4を配分する
3. Dropboxを同期フォルダーとして指定しない

## 1. NFSを利用する

vagrantを利用する際にホストマシンとゲストマシンでファイル同期(shared folder)を行なっていましたが、 
この設定がvagrantのパフォーマンスに影響する可能性があるようです。


> In some cases the default shared folder implementations (such as VirtualBox shared folders) have high performance penalties. If you are seeing less than ideal performance with synced folders, NFS can offer a solution.


こちらを解決するために同期方法にNFSを利用することにしました。

{% highlight ruby %}
  config.vm.synced_folder "/folder/path/on/host", "/folder/path/on/guest", nfs: true
{% endhighlight %}

しかしながら、今回のケースでは当初の問題が解決することはありませんでした。

## 2. システムメモリの1/4をゲストマシンに配分する

次に、[こちら](https://stefanwrobel.com/how-to-make-vagrant-performance-not-suck)の記事を参考に  
ゲストマシンに割り当てるメモリをシステムメモリの1/4に増加しました。  
下記の内容をVagrant ファイルに追記しました。

{% highlight ruby %}
    # Customize the amount of memory on the VM:
      host = RbConfig::CONFIG['host_os']
    # Give VM 1/4 system memory
    if host =~ /darwin/
      # sysctl returns Bytes and we need to convert to MB
      mem = `sysctl -n hw.memsize`.to_i / 1024
    elsif host =~ /linux/
      # meminfo shows KB and we need to convert to MB
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i
    elsif host =~ /mswin|mingw|cygwin/
      # Windows code via https://github.com/rdsubhas/vagrant-faster
      mem = `wmic computersystem Get TotalPhysicalMemory`.split[1].to_i / 1024
    end

    mem = mem / 1024 / 4
    vb.customize ["modifyvm", :id, "--memory", mem]
{% endhighlight %}

しかしながら、この対応でも問題は解決しませんでした。

## 3. Dropboxを同期フォルダーとして指定しない


上記1,2を対応してもCPUの占有率は100%のままでした。  
次に、
**ホストマシンのファイル同期先がDropbox上だったのをやめました。**   
これが効果てきめんで、嘘のようにCPUの占有率が下がりました。
このケースでは効果があっただけかもしれませんが、同様の症状で困っている方は試してみてもいいかもしれません。

ref 
[How to make Vagrant performance not suck](https://stefanwrobel.com/how-to-make-vagrant-performance-not-suck)




