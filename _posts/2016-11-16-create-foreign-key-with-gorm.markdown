---
layout: post
title:  "Gormで外部キーを設定する"
date:   2016-11-16 00:00:00 +0900
categories: go
---

GOでwebアプリケーションを作ることになり、ORマッパーにgormを採用したのですが、外部キーの設定についてメモ。

## Gormを利用して外部キーを設定する

外部キーを設定するにあったって下記の手順を行いました。

### 1. Structでデータを定義する

UserがCreditCardをHasOneで持っているAssociationを定義します。
CreditCardのUserIDがFKになる予定です。

{% highlight go %}
type User struct {
  gorm.Model
  Name string	
}

type CreditCard struct {
  gorm.Model
  User 	  User
  UserID 	int
  Number  string
}
{% endhighlight %}

この状態で、Migrateをしてみます。

{% highlight go %}
db.AutoMigrate(&User{}, &CreditCard{})
{% endhighlight %}

実際にテーブルのindexがどうなったか確認すると。


{% highlight sql %}
SELECT 
  TABLE_NAME, 
  COLUMN_NAME, 
  CONSTRAINT_NAME, 
  REFERENCED_TABLE_NAME, 
  REFERENCED_COLUMN_NAME  
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE 
WHERE TABLE_NAME = 'credit_cards'
{% endhighlight %}

```
Empty set (0.00 sec)
```

テーブルには外部キーが貼られてないようです。


### 2. AddForeignKeyを利用する

こちらの[issue](https://github.com/jinzhu/gorm/issues/23)を見つけました。 
実際にFKを貼るのであれば、`AddForeignKey関数`を利用すればいいとのこと。

AddForeignKeyは下記のような引数を取ります。

{% highlight go %}
func (s *DB) AddForeignKey(field string, dest string, onDelete string, onUpdate string) *DB {
  ・・・
}
{% endhighlight %}

上記を参考に、先程のmigrate処理を変更します。


{% highlight go %}
db.AutoMigrate(&User{})
db.AutoMigrate(&CreditCard{}).AddForeignKey("user_id", "users(id)", "RESTRICT", "RESTRICT")`
{% endhighlight %}

### 3. Error 1215の場合はdata型が一致していないので調整する

今回の場合、credit_cardsのuser_idの型指定をunsigned intにしていなかったので、参照カラムとのデータ型違いによりError 1215で外部キーを設定できませんでした。
定義を見直し、下記のように変更して調整しました。

*(int > uintに変更するだけで問題ないかもしれません)*


{% highlight go %}
type CreditCard struct {
  gorm.Model
  User 	  User
  UserID 	int   `gorm:"type:INT(10) UNSIGNED; not null"`
  Number  string
}
{% endhighlight %}

再度マイグレーションを実行して確認すると、
問題なく貼れました!

{% highlight sql %}
+--------------+-------------+---------------------------------------+-----------------------+------------------------+ 
| TABLE_NAME   | COLUMN_NAME | CONSTRAINT_NAME                       | REFERENCED_TABLE_NAME | REFERENCED_COLUMN_NAME | 
+--------------+-------------+---------------------------------------+-----------------------+------------------------+ 
| credit_cards | id          | PRIMARY                               | NULL                  | NULL                   | 
| credit_cards | user_id     | credit_cards_user_id_users_id_foreign | users                 | id                     | 
+--------------+-------------+---------------------------------------+-----------------------+------------------------+
{% endhighlight %}

参考 
 
* [AutoMigrate foreign keys #450](https://github.com/jinzhu/gorm/issues/450)  
* [Create foreign key constraint between references #23](https://github.com/jinzhu/gorm/issues/23)



