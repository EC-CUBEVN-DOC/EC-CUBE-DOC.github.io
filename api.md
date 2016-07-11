---
layout: default
title: API developmemt Guideline
---

---

This is document which summarized Guideline for developing API in EC-CUBE

In EC-CUBE 3

* Implement API handling based on principle of REST
* In REST, unique URI will be given for each resource which is called product list,　conduct operation such as submit and delete by sending parameter by HTTP method which is called GET and POST, PUT, DELETE with that URI.

#### relevant page
[Web API Authorization ガイド](/api_authorization.html)  
[EC-CUBE API β版 プラグイン スタートアップガイド](/web-api-doc.html)

## End point of API

Base URI of End Point will set https://ドメイン名/api/v1 , after that descrive path of each End point.

```
Ex) In case of /products
https://ドメイン名/api/v1/products
```


## API versioning

Include version in URL

```
http://ドメイン名/api/v1/products
```

In future, in case version up API, part of version of  `/v1` will be change, and API of old version will leave in order to maintain compatibility.
Version is just interger, do not create minor version.


## resource name

REST APIではURLでリソースを表現し、そのリソースへの操作をHTTPメソッドを用いて表現します。

resource example)

|目的|URL|HTTPメソッド|
|---|---|---|
|商品一覧の取得|/products|GET|
|商品の新規登録|/products|POST|
|商品情報の取得|/products/:id|GET|
|商品情報の更新|/products/:id|PUT|
|商品情報の削除|/products/:id|DELETE|


* Just use lower case letter
* Display resource by name
* Put many basic concrete noun as name as much as possible
* Show operation of resource by HTTP method
* In order to be easy to understand, do not use deeply URL more than `/products/:id/xxxxx`.
* URLは浅く保ち複雑なものはクエリストリングにする。
* About Query string name, if transfer many by array, will set plural form; if just transfer a part, will set singular form.

ただし、RESTには必ずこだわらず利便性を重要視します。


## RESTに適さないAPI
検索といったリソース操作でないAPIの場合、名詞でなく動詞を使います。

```
例)
https://ドメイン名/api/v1/search?name=aaaa&price=1000



## error process

#### HTTPステータスコード
エラー発生時にHTTPステータスコードを使いますが、HTTPステータスコードは全て網羅せず以下に止めておきます。

|ステータスコード|目的|意味|
|---|---|---|
|200|OK|リクエストは成功し、レスポンスとともに要求に応じた情報が返される。|
|201|Created|リクエストの処理が完了し、新たに作成されたリソースのURIが返される。POSTやPUTの応答で利用する。|
|304|Not Modified|リクエストしたリソースは更新されていないことを示す。|
|400|Bad Request|定義されていないメソッドを使うなど、クライアントのリクエストがおかしい場合に返される。|
|401|Unauthorized|認証が必要である。Basic認証やDigest認証などを行うときに使用される。|
|403|Forbidden|リソースにアクセスすることを拒否された。アクセス権がない場合や、ホストがアクセス禁止処分を受けた場合などに返される。|
|404|Not Found|リソースが見つからなかった。|
|500|Internal Server Error|サーバサイドでエラーが発生した場合に返される。|
|503|Service Unavailable|サービス利用不可。サービスが一時的に過負荷やメンテナンスで使用不可能である。|

* 参考 [https://ja.wikipedia.org/wiki/HTTPステータスコード](https://ja.wikipedia.org/wiki/HTTPステータスコード)

EC-CUBE 3ではレスポンスを渡す時にHTTPステータスコードの`200`番台を返すようにします。

```php
$data = 'aaa';
return $app->json($data, 201);
```
ただし、正常系のGetメソッドに関してはHTTPステータスコードの`200`は渡しません。

```php
$data = 'aaa';
return $app->json($data);
```

#### エラー時のレスポンス
HTTPステータスコードに加えてエラーが発生した場合、エラーコード, エラーメッセージ, エラー詳細などをJSONレスポンスで返すようにします。
また、エラーが複数渡せるように配列にしておきます。

```json
{
  "errors": [
    {
      "code": 100,
      "message": "product not found."
    },
    {
      "code": 101,
      "message": "item not found."
    }
  ]
}

```
* 参考 [http://qiita.com/suin/items/f7ac4de914e9f3f35884](http://qiita.com/suin/items/f7ac4de914e9f3f35884)

エラーが発生したときのステータスコードは処理によって異なりますが、基本は`400`番台を返すようにします。  
また、レスポンスヘッダの内容については別途検討します。


## レスポンスヘッダについて
今後、認証処理のヘッダ情報などEC-CUBE 3独自のヘッダ内容を記述します。

## レスポンスのフォーマット
レスポンスデータフォーマットはJSONのみを原則とします。


## 返り値について
JSONの属性名に規約はありませんがJavaScriptの命名規約においてキャメルケースを使うケースが多いため、
なるべく先頭小文字のキャメルケースを使う方が望ましいですが、EC-CUBE 3にとって使い勝手の良い形式とするため特に制約は設けません。

ただし、JSONの返り値の形式は必ず**key-value形式**にします。

```json
// サンプル
{
  "menu": {
    "id": "file",
    "value": "File",
    "popup": {
      "menuitem": [
        {"value": "New", "onclick": "CreateNewDoc()"},
        {"value": "Open", "onclick": "OpenDoc()"},
        {"value": "Close", "onclick": "CloseDoc()"}
      ]
    }
  }
}


{
  "id": 1,
  "name": "a",
  "photoUrls": [],
  "tags": [],
  "status": "available"
}
```
* 参考 [http://json.org/example.html](http://json.org/example.html)



#### データ型
1. Date型  
日付データの形式にはRFC 3339を用います。また、時差対応しやすくするためUTCで返すことを原則とします。  
例：2014-08-30T20:00:00Z

1. boolean型  
true、falseを返します。

1. 数値型  
文字列に変換するのではなく、数値のままで返すようにします。  
ただし金額の場合、金額の値については文字列として表現("1000")します。

1. 文字列
""で囲んで文字列を返します。  
タブや改行など、いくつかの特殊な文字はエスケープする必要があります。


1. null扱い  
nullとして値がセットされていた場合、空文字などに変換せずそのまま返します。




## URLパラメータ名
URLパラメータ名はEntityのプロパティ名を基本的に利用します。(DBの項目名とは異なるプロパティ名もあります。)
また、検索画面については以下の共通パラメータ指定を行います。

#### ページネーション
`limit` と `offset` パラメータで指定することで、`offset`番目から`limit`件取得できるようにします。  
例) `/products?limit=25&offset=50`

戻り値のJSONには全レコード件数を metadata としてレスポンスに含めるようにします。
省略時のデフォルト件数はデータサイズやアプリケーションによって決定します。

```json
{
  "products" : [
    // -- 省略  --
  ],
  "metadata" : [
    {
      "totalItemCount" : 119,
      "limit" : 15,
      "offset" : 41
    }
  ]
}
```

#### フィールド指定
レスポンス量を増やさないために、フィールドを指定するとそのフィールドの値だけを返すように制御します。  
例) `/products?fields=name,color,location`

`fields` パラメータにカンマ区切りで指定することで指定したフィールドのみを返します。


## パラメータチェックについて
FormTypeを利用できる箇所はFormTypeを使って入力チェックを行い、利用できない箇所は個別にチェックを行います。  
個別入力チェックについては`Symfony\Component\Validator\Constraints`パッケージにあるクラスを極力使うようにします。

###### 個別入力チェックサンプル

 
<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/api/SampleValidate.php"></script>

## 認証について

EC-CUBE で Web API を実行する際、一般公開された情報を参照する場合は必要ありませんが、顧客情報を参照したり、受注情報を更新する場合などは認証が必要です。

EC-CUBE 3 では、 OpenID Connect を使用した

[OAuth2.0 Authorization](http://openid-foundation-japan.github.io/rfc6749.ja.html) 及び [OpenID Connect](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html) をサポートしています。

詳しくは [Web API Authorization ガイド](/api_authorization.html) を参照してください。

### 対応する認証フロー

以下の認証フローに対応しています。

- [OAuth2.0 Authorization Code Flow](http://openid-foundation-japan.github.io/rfc6749.ja.html#grant-code) - 主にWebアプリ向け
- [OAuth2.0 Implicit Flow](http://openid-foundation-japan.github.io/rfc6749.ja.html#grant-implicit) - 主にJavaScript、 ネイティブアプリ向け
- [OpenID Connect Authorization Code Flow](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#CodeFlowAuth) - 主にWebアプリ向け
- [OpenID Connect Implicit Flow](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#ImplicitFlowAuth) - 主にJavaScript、 ネイティブアプリ向け

### 利用方法

#### 管理画面メンバー(Member)

1. 管理画面→設定→システム情報設定→メンバー管理→メンバーの編集より **APIクライアント一覧** をクリックします。
2. 「新規作成」より、APIクライアントを新規登録します。
    - **アプリケーション名** には任意の名称を入力します
    - **redirect_uri** には、Authorization Endpoint からのリダイレクト先の URL を入力します。ネイティブアプリやテスト環境用に `urn:ietf:wg:oauth:2.0:oob` を使用することも可能です。
3. 登録が終わると、`client_id`, `client_secret` などが発行されます。公開鍵は `id_token` を検証する際に使用します。
3. APIクライアントを実装します。

#### 会員(Customer)

1. mypage にログインし、 `/mypage/api` へアクセスします。
2. **新規登録** をクリックし、 APIクライアントを新規登録します。
    - **アプリケーション名** には任意の名称を入力します
    - **redirect_uri** には、Authorization Endpoint からのリダイレクト先の URL を入力します。
3. 登録が終わると、 `client_id`, `client_secret` などが発行されます。公開鍵は `id_token` を検証する際に使用します。

### サンプルクライアント

- [PHP(Symfony2) での実装例](https://github.com/nanasess/eccube3-oauth2-client)
- [Python(Flask) での実装例](https://github.com/nanasess/eccube3-oauth2-client-for-python)
- [Node.js(Express) での実装例](https://github.com/nanasess/eccube3-oauth2-client-for-nodejs)
- [C# での実装例(Web/Wpf)](https://github.com/nanasess/DotNetOpenAuth)
- [Java での実装例](https://github.com/nanasess/eccube3-oauth2-client-for-java)
- [Google OAuth 2.0 Playground](https://developers.google.com/oauthplayground/)
    - OAuth 2.0 Configuration -> OAuth endpoint -> *Custom* にて動作確認済み
    - Authorization Endpoint に `?state=<random_state>` を付与する必要があります


## ドキュメント
Swagger Editorを使ってWeb APIドキュメント(swagger.yml)を記述します。

[Swagger Editor](http://editor.swagger.io/)

* 参考 [http://qiita.com/weed/items/539f6bbade6b75980468](http://qiita.com/weed/items/539f6bbade6b75980468)


## 参考URL
この指針は以下のサイトを参考にさせていただきました。

[これから始めるエンタープライズ  Web API 開発](https://www.ogis-ri.co.jp/otc/hiroba/technical/WebAPI/part2.html)  
[Web API設計指針を考えた](http://blog.mmmcorp.co.jp/blog/2015/07/01/web_api_guideline/index.html)  
[Web API 設計のベストプラクティス集 Web API Design - Crafting Interfaces that Developers Love](http://d.hatena.ne.jp/cou929_la/20130121/1358776754)

