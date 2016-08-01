---
layout: post
title:  "画像をhttp経由でダウンロードする"
date:   2016-08-01 17:00:00 +0900
categories: ruby
---

rubyでhttp経由で画像をダウンロードするメモ

{% highlight ruby %}

def download_image(dest, image_url)
  File.open(dest, 'wb') do |file|
    file.write(open(image_url).read)
  end
end

{% endhighlight %}




