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

However, in REST, emphasis convience without regard.


## API tthat is not suitable for REST
In case of API that is not resource operation which is called search, use verb, not noun.

```
Ex)https://ドメイン名/api/v1/search?name=aaaa&price=1000



## error process

#### HTTP status code 
When occurred error, use HTTP status code, but HTTP status code will not cover all, stop as below

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

* Reference  [https://ja.wikipedia.org/wiki/HTTPステータスコード](https://ja.wikipedia.org/wiki/HTTPステータスコード)

EC-CUBE 3ではレスポンスを渡す時にHTTPステータスコードの`200`番台を返すようにします。

```php
$data = 'aaa';
return $app->json($data, 201);
```
However, relating to Get method of normal system, do not transfer `200` of HTTP status code
 

```php
$data = 'aaa';
return $app->json($data);
```

#### Response of when was error 
In case added in HTTP status code, and occurred error, make sure that return error code, error message, error detail by JSON response.
And, set array in order to transfer many errors.

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

Status code when occurred error, will be different based on process. However basically, try to return `400` series.   
However, contents of respose header will consider separately.


## About response header
In future, describe unique header contents of EC-CUBE 3 such as header info to authentication process

## Format of response
As generalrule, response data format will be just JSON.


## About return value
There is no rule in property name of JSON, but there are many cases which use camel case for naming rule of JavaScript
so that Use of camel case of the first lower character is expected. In EC-CUBE 3, there is not speial rule in order to tcreate user-friendly format.

However, format of returned value will choose **key-value形式**

```jsonile",
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



#### Data type
1. Date type  
In format of date data, use RFC 3339. As general rule, return by UTC in order to handle for  difference in time
Ex) 2014-08-30T20:00:00Z

1. boolean type  
Return true, false.

1. numeric type  
Do not convert into character string, just remain numeric value and return.
But, in case of amount of money, display("1000") value of amount of money by character string.

1. character string
Surround by "" and return character sting  
It is neccessary to escape Tab, lines breaks, some special texts.


1. Use of null  
In case value is set as, just leave it and return without converting into empty text.




## URL parameter name
Basically, URL parameter name will use property name of Entity (Sometimes Item name of DB will be different with Property name)
And, about searching screen, execute the following common paramter specification.

#### Pagination
By specifying parameter of `limit`and `offset`, make sure that can get `limit` from `offset`  
Ex)  `/products?limit=25&offset=50`

In JSON of returned value, make sure that include all of records in Response as metadata.
The number of default records when omitting, will decide based on Data size and Application

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

#### Specify Field
Response volume will not increase, so if specify field, control in order to return just that field  
例) `/products?fields=name,color,location`

Return only the specified field by specifying in parameter `fields` by Comma Separated Value


## About check paramter
ABout place that can use FormType will use FormType to check input, place which can not use, will check separately.  
ABout individual check will use class which exist in package `Symfony\Component\Validator\Constraints` as much as possible

###### Individual input check sample

 
<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/api/SampleValidate.php"></script>

## About Authentication

IN EC-CUBE, when execute Web API, there is no need in case refer the general public. However it is necessary in case refer the customer info or update the receiving order info.

In EC-CUBE 3, OpenID Connect was used

Supporting [OAuth2.0 Authorization](http://openid-foundation-japan.github.io/rfc6749.ja.html) and [OpenID Connect](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html) 

Detail will refer  [Web API Authorization ガイド](/api_authorization.html)

### The handling authentication flow

Handling for the following authentication flow

- [OAuth2.0 Authorization Code Flow](http://openid-foundation-japan.github.io/rfc6749.ja.html#grant-code) - 主にWebアプリ向け
- [OAuth2.0 Implicit Flow](http://openid-foundation-japan.github.io/rfc6749.ja.html#grant-implicit) - 主にJavaScript、 ネイティブアプリ向け
- [OpenID Connect Authorization Code Flow](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#CodeFlowAuth) - 主にWebアプリ向け
- [OpenID Connect Implicit Flow](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#ImplicitFlowAuth) - 主にJavaScript、 ネイティブアプリ向け

### Usage method

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


## Document
Use Swagger Editor to describe Web API document (swagger.yml)

[Swagger Editor](http://editor.swagger.io/)

* 参考 [http://qiita.com/weed/items/539f6bbade6b75980468](http://qiita.com/weed/items/539f6bbade6b75980468)


## Reference URL
この指針は以下のサイトを参考にさせていただきました。

[これから始めるエンタープライズ  Web API 開発](https://www.ogis-ri.co.jp/otc/hiroba/technical/WebAPI/part2.html)  
[Web API設計指針を考えた](http://blog.mmmcorp.co.jp/blog/2015/07/01/web_api_guideline/index.html)  
[Web API 設計のベストプラクティス集 Web API Design - Crafting Interfaces that Developers Love](http://d.hatena.ne.jp/cou929_la/20130121/1358776754)

