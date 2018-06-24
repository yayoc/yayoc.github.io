---
layout: post
title:  "swagger-to-mockでswaggerファイルからmock jsonを作成する"
date:   2018-06-24 00:00:00 +0900
categories: ts
---

API のドキュメントツールは依然として、`swagger`を使うことが多いのですが、  
Unit Tests を書くときにmockデータを swagger から簡単に書き出せるといいなと思うことがあります。

そこで、今回[swagger-to-mock](https://github.com/yayoc/swagger-to-mock)という mock データを生成するCLIを作りました。  
swagger で定義した yaml ファイルをパースして、`example`データが有れば、example の値を。そうでなければ、下記のリストにある各型で定義したデフォルトの値を含む json ファイルを吐き出してくれます。

```yaml
string: ""
number: 0
integer: 0
boolean: true
array: []
object: {}
```

`$npx swagger-to-mock swagger.yaml`

としてあげると、定義された各レスポンスに対して、`${API_PATH}_${HTTP_METHOD}_${RESPONSE_STATUS}.json` の命名規則に基づき、json ファイルが生成されます。

例えば、下記のような内容の swagger ファイルに対して、`swagger-to-mock`を実行すると

```yaml
responses:
  '200':
    description: pet response
    content:
      application/json:
        schema:
          type: array
          items:
            $ref: '#/components/schemas/Pet'
components:
  schemas:
    Pet:
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
          example: 123
        name:
          type: string
          example: "Tom"
        tag:
          type: string
          example: "Cat"
```

`pets_get_200.json`というファイルが生成され、ファイルの中身は下記のようになります。

```json
[
  {
    "id": 123,
    "name": "Tom",
    "tag": "Cat"
  }
]
```

現状、swagger3 (OpenAPI3) にしか対応していませんが、必要に応じて、swagger2 のサポートも追加したいと考えています。  
また、各データタイプの format や pattern を考慮することで、より意味のあるデータ生成を行える機構も検討したいと考えています。

参考

- [swagger-to-mock](https://github.com/yayoc/swagger-to-mock)
