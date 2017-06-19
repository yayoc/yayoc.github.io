---
layout: post
title:  "swagger-to-flowtypeでswaggerファイルからFlow type aliasを作成する"
date:   2017-06-11 00:00:00 +0900
categories: js
---

APIのドキュメントツールとして、`swagger`を使う場面があり、  
そもそも定義されたオブジェクト数も多く、
また、ネストが深いオブジェクトもある中で、[Flow](https://flow.org/)における[Type ailias](https://flow.org/en/docs/types/aliases/)を手動で作成するのは、なかなか厳しいものがありました。  

そこで、今回npm moduleとして**[swagger-to-flowtype](https://github.com/yayoc/swagger-to-flowtype)**というコマンドラインツールを作りました。  
定義されたyamlもしくは、jsonファイルをパースして、definitionsを抽出、type aliasとして定義、ファイルの書き出しを行います。  

例えば、下記のような定義をしたswagger.yamlを、

```yaml
...

definitions:
  Order:
    type: "object"
    properties:
      id:
        type: "integer"
        format: "int64"
      petId:
        type: "integer"
        format: "int64"
      quantity:
        type: "integer"
        format: "int32"
      shipDate:
        type: "string"
        format: "date-time"
      status:
        type: "string"
        description: "Order Status"
        enum:
        - "placed"
        - "approved"
        - "delivered"
      complete:
        type: "boolean"
        default: false
    xml:
      name: "Order"
  Category:
    type: "object"
    properties:
      id:
        type: "integer"
        format: "int64"
      name:
        type: "string"
    xml:
      name: "Category"
...
```

`$swagger-to-flowtype swagger.yaml`  

としてあげると下記のように型を定義した`flowtype.js`というファイルが生成されます。  

```js
// @flow
export type Order = {
  id: number,
  petId: number,
  quantity: number,
  shipDate: string,
  status: string,
  complete: boolean
};
export type Category = { id: number, name: string };
```

swaggerでAPIドキュメントを書いている + `Flow`を使っている場合には使ってみてもいいかもしれません。  


参考  

* [swagger-to-flowtype](https://github.com/yayoc/swagger-to-flowtype)



