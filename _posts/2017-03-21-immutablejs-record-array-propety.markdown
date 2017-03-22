---
layout: post
title:  "ImmutableJS Recordでarray propertyをImmutableに扱う"
date:   2017-03-21 00:00:00 +0900
categories: JavaScript
---

`ImmutableJS`で下記の用にモデルを設計したとします。

```JavaScript
import { Record, fromJS } from 'immutable';

const UserRecord = Record({
  id: null,
  name: null,
  profile_image_url: null,
  bio: null,
  channels: [],
})
```

で、下記のようにUserRecordを生成した場合、 

```JavaScript
const user = new UserRecord({
  id: 1,
  name: "yayoc",
  profile_image_url: "",
  bio: "",
  channels: [
    {id: 1, name: "channel 1"},
    {id: 2, name: "channel 2"},
    {id: 3, name: "channel 3"},
  ],
});
```
userのchannelsを確認するとList型ではなくObjectの配列が返ります。  
Recordでは、配列のプロパティに関しては、Immutableに変換してくれません。

![Imgur](http://i.imgur.com/3uYzN91.png)

そこで、下記のようにUserRecordを継承したUser classを作成して、  
constuructorをオーバーライドすることで、配列に関してもImmutableに変更することが可能になります。

```JavaScript
import { Record, fromJS } from 'immutable';

const UserRecord = Record({
  id: null,
  name: null,
  profile_image_url: null,
  bio: null,
  channels: [],
})

class User extends UserRecord {
  constructor(values) {
    super(fromJS(values));
  }
}

```

```JavaScript
const user = new User({
  id: 1,
  name: "yayoc",
  profile_image_url: "",
  bio: "",
  channels: [
    {id: 1, name: "channel 1"},
    {id: 2, name: "channel 2"},
    {id: 3, name: "channel 3"},
  ],
});
```

![Imgur](http://i.imgur.com/HNKvLvj.png)


参考  

* [Record](https://facebook.github.io/immutable-js/docs/#/Record)

