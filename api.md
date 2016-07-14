---
layout: default
title: API developmemt Guideline
---

---

This is document which summarized Guideline for developing API in EC-CUBE

In EC-CUBE 3

* Implement API handling based on principle of REST
* In REST, unique URI will be given for each resource which is called product list,　conduct operation such as submit and delete by sending parameter by HTTP method which is called GET and POST, PUT, DELETE with that URI.

#### Relevant page
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

IN REST API, express resource by URL, use HTTP method to express the operation to that resource.

Resource example)

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
* URLは浅く保ち複雑なものはクエリストリングにする。(※)
* About Query string name, if transfer many by array, will set plural form; if just transfer a part, will set singular form.

However, in REST emphasis the convience without regarding


## API that is not suitable for REST
In case of API that is not resource operation which is called search, use verb, not noun.

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

* Reference  [https://ja.wikipedia.org/wiki/HTTPステータスコード](https://ja.wikipedia.org/wiki/HTTPステータスコード)

In EC-CUBE 3, make sure that return `200` series of HTTP status code when transfer response.

```php
$data = 'aaa';
return $app->json($data, 201);
```
However, relating to Get method of normal system, do not transfer `200` of HTTP status code.

```php
$data = 'aaa';
return $app->json($data);
```

#### Response when occurred error
In case add into HTTP status code, and occurred error, make sure that return error code, error message, error detail by JSON response
Morever, set array in order to transfer many error.

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
As general rule, response data format will be just JSON.


## About return value
There is no rule in property name of JSON, but there are many cases which use camel case for naming rule of JavaScript
so that Use of camel case of the first lower character is expected. In EC-CUBE 3, there is not speial rule in order to tcreate user-friendly format.

However, format of returned value will choose **key-value形式**

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



#### Data type
1. Date type    
In format of date data, use RFC 3339. As general rule, return by UTC in order to hanとします。  
Ex) 2014-08-30T20:00:00Z

1. boolean type  
Return true, false.


1. Numeric value  
Remain numeric value and return without convert character string.  
But in case of amount of money, display ("1000") value of amount of money as character string 

1. Character string
Surround in "" and return character string.  
It is necessary to escape Tab, lines breaks, some special characters.


1. Handle null  
In case value is set as null, remain like that and return without converting into null text.




## URL parameter name
URL parameter name will use property name of Entity. (Sometimes, there are property names which are different with Item name od DB)
Morever, about searching screen, conduct specifying the following common parameter.

#### Pagination
By specifying parameter `limit` and `offset`, try to get record `limit` from `offset` .  
例) `/products?limit=25&offset=50`

In JSON of returned value, include all records into response as metadata.
The number of default records when omitted, will decide based on data size and Application.

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

#### Specify field
Response volume will not increase, if specify field, control in order to return just value of that field.  
Ex) `/products?fields=name,color,location`

When specify in paramter `fields` by comma separated value, return just the specified field.


## About parameter check
Place that can use FormType will use FormType to check input, place that can not use, will check separately.  
About individual input check, try to use class that exists in package `Symfony\Component\Validator\Constraints` as much as possible

###### Individual input check sample

 
<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/api/SampleValidate.php"></script>

## Authentication

In EC-CUBE, when excecute Web API, it is not necessary in case refer the general public info. But it is necessary for authentication in case refer the customer info or update the receiving order info.

In EC-CUBE 3, OpenID Connect is used.

Supporting [OAuth2.0 Authorization](http://openid-foundation-japan.github.io/rfc6749.ja.html) and  [OpenID Connect](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html).

Please refer detail at [Web API Authorization ガイド](/api_authorization.html) 

### The handled authentication flow

Handling for the following authentication flow


- [OAuth2.0 Authorization Code Flow](http://openid-foundation-japan.github.io/rfc6749.ja.html#grant-code) - For Web App
- [OAuth2.0 Implicit Flow](http://openid-foundation-japan.github.io/rfc6749.ja.html#grant-implicit) - For JavaScript , Native App
- [OpenID Connect Authorization Code Flow](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#CodeFlowAuth) - For Web App
- [OpenID Connect Implicit Flow](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#ImplicitFlowAuth) - For JavaScript, Native App

### Usage method

#### 管理画面メンバー(Member)

1. Click 管理画面(management screen)→設定(setting)→システム情報設定(system info setting)→メンバー管理(member management)→ **APIクライアント一覧**(API client list) from editing member
2.  Register new APOI client from [新規作成](new create)
    -  Input the optional name for **アプリケーション名** 
    - In **redirect_uri**, input URL of direct destination from Authorization Endpoint. It is possible to use `urn:ietf:wg:oauth:2.0:oob` for Native App and Testing environment.
3. If the registration has finished, `client_id`, `client_secret` will be issued. The public key will use when verify  `id_token` 
3. Execute API Client

#### 会員(Customer)

1. Login in mypage, access to `/mypage/api` 
2. Click **新規登録**, register new  API client 
    - Input the optional name for  **アプリケーション名**
    - In **redirect_uri** , input URL of direct destination from Authorization Endpoint
3. If the registration has finished, `client_id`, `client_secret` will be issued. The public key will use when verify  `id_token`

### Sample client

- [PHP(Symfony2) での実装例](https://github.com/nanasess/eccube3-oauth2-client)
- [Python(Flask) での実装例](https://github.com/nanasess/eccube3-oauth2-client-for-python)
- [Node.js(Express) での実装例](https://github.com/nanasess/eccube3-oauth2-client-for-nodejs)
- [C# での実装例(Web/Wpf)](https://github.com/nanasess/DotNetOpenAuth)
- [Java での実装例](https://github.com/nanasess/eccube3-oauth2-client-for-java)
- [Google OAuth 2.0 Playground](https://developers.google.com/oauthplayground/)
    - Confirmed already operation in OAuth 2.0 Configuration -> OAuth endpoint -> *Custom* 
    - It is necessary to give  `?state=<random_state>` for Authorization Endpoint.


## Document
Use Swagger Editor to describe Web API document (swagger.yml)

[Swagger Editor](http://editor.swagger.io/)

* Reference  [http://qiita.com/weed/items/539f6bbade6b75980468](http://qiita.com/weed/items/539f6bbade6b75980468)


## Reference URL
This guidline will refer the following site

[これから始めるエンタープライズ  Web API 開発](https://www.ogis-ri.co.jp/otc/hiroba/technical/WebAPI/part2.html)  
[Web API設計指針を考えた](http://blog.mmmcorp.co.jp/blog/2015/07/01/web_api_guideline/index.html)  
[Web API 設計のベストプラクティス集 Web API Design - Crafting Interfaces that Developers Love](http://d.hatena.ne.jp/cou929_la/20130121/1358776754)

