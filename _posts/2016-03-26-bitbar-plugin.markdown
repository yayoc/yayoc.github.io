---
layout: post
title:  "bitbar pluginを作ってみた"
date:   2016-03-26 09:09:53 +0900
categories: bitbar
---

[Rebuild.fm](https://rebuild.fm/132a/)でも紹介のあった[bitbar](https://github.com/matryer/bitbar)のプラグインを作成してみました。

[https://getbitbar.com/plugins/Music/songkick.1d.py](https://getbitbar.com/plugins/Music/songkick.1d.py)

![Songkick plugin capture](http://i.imgur.com/pu8gJh2.png)

[Songkick](http://www.songkick.com/)という音楽イベント情報を提供しているサービスのAPIを利用しています。  
IPを元に、近くで予定されているイベントを表示できます。  
ソースコードは下記です。

{% highlight python %}

#!/usr/bin/env python

import json
import urllib2

def get_events():
    try:
        ip = get_my_ip()
        request = urllib2.Request('http://api.songkick.com/api/3.0/events.json?apikey=' + configs['apikey'] + '&location=ip:' + ip)
        response = urllib2.urlopen(request)
        response = json.load(response)
        return map(format_events, response['resultsPage']['results']['event'])
    except Exception:
        print ':('

def format_events(event):
    return (u'%s / %s | href=%s' % (event['displayName'], event['location']['city'], event['uri'])).encode('utf-8')

def get_my_ip():
    request = urllib2.Request('http://ip.42.pl/raw')
    response = urllib2.urlopen(request)
    return response.read()

configs = {
    'apikey': 'Your API Key'
}

print (u'\u24c8 | size=18').encode('utf-8')
print '---'
print '\n'.join(get_events())
print '---'
print 'Songkick - Upcoming events | href=https://songkick.com'

{% endhighlight %}


ただ、このプラグイン使用するにはAPI Keyが必要なため、
[こちら](http://www.songkick.com/api_key_requests/new)から申請が必要です。


[@miyagawa](https://twitter.com/miyagawa)さんも言ってたように、
数行のコードでちょっとしたプラグインが作れるのは、エンジニアの好奇心くすぐるなと思います。

P.S. 
ちなみにこのプラグインを使って、僕はしっかり[Tortoise](http://www.songkick.com/artists/136413-tortoise)のライブチケットを購入しました

